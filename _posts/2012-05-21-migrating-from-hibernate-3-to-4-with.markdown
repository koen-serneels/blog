---
layout: post
title: Migrating from Hibernate 3 to 4 with Spring integration
date: '2012-05-21T13:57:00.000+02:00'
author: Koen Serneels
img: 2.png
tags:
- Hibernate
modified_time: '2012-07-03T13:25:44.209+02:00'
blogger_id: tag:blogger.com,1999:blog-3406746278542965235.post-3830300551371174590
blogger_orig_url: http://koenserneels.blogspot.com/2012/05/migrating-from-hibernate-3-to-4-with.html
---

This week it was time to upgrade our code base to the latest Hibernate 4.x.
We postponed our migration (still being on Hibernate 3.3) since the newer maintenance releases of the 3.x branch required some API changes which were apparently still in flux. An example is the UserType API which was still showing flaws and was going to be finalized in Hibernate 4.

The migration went quite smooth. Adapting the UserType's to the new interface was pretty straightforward. There were some hick ups here and there but nothing painful.

The thing to watch out for is the Spring integration. If you have been using Spring with Hibernate before, you will be using the <b>LocalSessionFactoryBean</b> (or <b>AnnotationSessionFactoryBean</b>) for creating the <b>SessionFactory</b>. For hibernate 4 there is a separate one in its own package: org.springframework.orm.<b>hibernate4</b> instead of org.springframework.orm.<b>hibernate3</b>. The <b>LocalSessionFactoryBean</b> from the hibernate 4 package will do for both mapping files as well as annotated entities, so you only need one for both flavors.

When the upgrade was done, all our tests were running and the applications were also running fine on Tomcat using the local Hibernate transaction manager. However when running on Glassfish using JTA transactions (and Spring's <b>JtaTransactionManager</b>) we got the <b>"No Session found for current thread"</b> when calling sessionFactory.getCurrentSession();

So it seemed I missed something in relation with the JTA configuration. As you normally do with the Spring-Hibernate integration, you let Spring drive the transactions. You specify a transaction manager and Spring makes sure all resources are registered with the transaction manager and eventually call commit or rollback. Spring will integrate with Hibernate, so it makes sure the session is flushed before a transaction commit.

When using hibernate 3 and the hibernate 3 Spring integration, the session is bound to a thread local. This technique allow you to use the <b>sessionFactory.getCurrentSession()</b> to obtain a open session anywhere inside the active transaction. This is both the case for the local <b>HibernateTransactionManager</b> as for the <b>JtaTransactionManager</b>.

However, as of the hibernate 4 integration, the hibernate session will be bound to the currently running JTA transaction instead.

From a user point of view nothing changes as <b>sessionFactory.getCurrentSession()</b> will still do its job. But when running JTA this means that Hibernate must be able to lookup the transaction manager to able to register the session with the currently running transaction. This is new if you are coming from Hibernate 3 with Spring, in fact, you did not have to configure anything in regards to transactions in your Hibernate <b>SessionFactory</b> (or <b>LocalSessionFactoryBean</b>) configuration.
As it turned out, with the Hibernate 4 Spring integration the transaction manager lookup configuration is effectively done by hibernate and not by Spring's <b>LocalSessionFactoryBean</b>.
The solution was pretty simple; adding this to the Hibernate (<b>LocalSessionFactoryBean</b>) configuration solved our problems:

<pre class="brush: xml;">
&lt;prop key="hibernate.transaction.jta.platform">org.hibernate.service.jta.platform.internal.SunOneJtaPlatform&lt;/prop&gt;
</pre>

<b>"SunOneJtaPlatform"</b> should then be replaced by a subclass that reflects your container.
See the <a target="_blank" href="http://docs.jboss.org/hibernate/orm/4.1/javadocs/org/hibernate/service/jta/platform/internal/AbstractJtaPlatform.html">API docs</a> for the available subclasses.
What this class does is actually telling Hibernate how it can lookup the transaction manager for your environment. If you don't configure this there will be nothing for Hibernate to bind the session to hence throwing the exception. 

There is also a property: 
<pre class="brush: xml;">
hibernate.current_session_context_class
</pre>

Which should point to <b>org.springframework.orm.hibernate4.SpringSessionContext</b>, but this automatically done by the <b>LocalSessionFactoryBean</b>, so there is no need to specify it in the configuration.

As this solved my "No Session found for current thread" problem, there was still another one. Changes made to the database inside a transaction were not visible after a successful transaction commit. After some research I found out that no one was calling <b>session.flush()</b>. While with the hibernate 3 integration there was a <b>SpringSessionSynchronization</b> registered which would call <b>session.flush()</b> prior transaction commit (in the beforeCommmit method).

In the hibernate 4 integration there is a <b>SpringFlushSynchronization</b> registered, which, as its name says, will perform a flush also. However, this is only implemented in the actual “flush” method of the <b>TransactionSynchronization</b>, and this method gets never called.

I raised an <a target="_blank" href="https://jira.springsource.org/browse/SPR-9404">issue</a> for this on Spring bugtracker, including  two sample application which illustrates the problem clearly.
The first uses Hibernate 3 and the other is the exact same application but this time using  hibernte 4. The second will show that no information is actually persisted to database (both apps are tested under the latest Glassfish 3.1.2)

 Until now the best workaround seems to be creating a flushing Aspect that wraps around <b>@Transactional</b> annotations. Using the order attribute you can order the transactional annotation to be applied before your flushing Aspect. This way your Aspect is still running inside the transaction, and is able to flush the session. It can obtain the session the normal way by injecting the <b>SessionFactory</b> (one way or the other) and then calling <b>sessionFactory.getCurrentSession().flush()</b>.

<pre class="brush: xml;">
&lt;tx:annotation-driven order="1"/&gt;

&lt;bean id="flushinAspect" clas="..."&gt;
 &lt;property name="order" value="2"/&gt;
&lt;/bean&gt;
</pre>

or, if using the annotation configuration:

<pre class="brush: java;">
@EnableTransactionManagement(order=1)
</pre>

<b><u>Update:</u></b>
There was some feedback on the issue. As it turns out it does not seem to be a bug in the Spring Hibernate integration, but a missing Hibernate configuration element. Apparently the "hibernate.transaction.factory_class" needs to be set to JTA, the default is JDBC which depends on the Hibernate Transaction API for explicit transaction management. By setting this to JTA the necessary synchronizations are registered by hibernate which will perform the flush. See the Spring <a target="_blank" href="https://jira.springsource.org/browse/SPR-9404">https://jira.springsource.org/browse/SPR-9404</a><p/><b><u>Update 2:</u></b>
As it turns out, after correcting the configuration as proposed on the preceding issue, there was still a problem. I'm not going to repeat everything, you can find detailed information in the second bug entry I submitted here: <a target="_blank" href="https://jira.springsource.org/browse/SPR-9480">https://jira.springsource.org/browse/SPR-9480</a>
It basically comes down to the fact that in a JTA scenario with the JtaTransactionFactory configured, hibernate does not detect that it is in a transaction and will therefore not execute intermediate flushes. With the JtaTransactionFactory configured, you are expected to control the transaction via the Hibernate API rather then via an external (Spring in our case) mechanism. One of the side effects is that you might be reading stale data in some cases.<p/> Example:

<pre class="brush: java;">
//[START TX1]
Query query = session.createQuery("from Person p where p.firstName = :firstName and p.lastName = :lastName");
Person johnDoe = (Person)query.setString("firstName","john").setString("lastName","doe").uniqueResult();
johnDoe.setFirstName("Jim");
Person jimDoe = (Person)query.setString("firstName","jim").setString("lastName","doe").uniqueResult();
//[END TX1]
</pre>

What happens is that when performing the second query at line 5, hibernate should detect that it should flush the previous update which was made to the attached entity on line 4 (updating the name from 'john' to 'jim').

However, because hibernate is not aware it is running inside an active transaction, the intermediate flushing doesn't work. It will only flush once before the transaction commits. This results in stale data, as the 2nd query would not find "jim" and return null instead. The solution (see the reply from Juergen Hoeller in the issue) is configuring <b>hibernate.transaction.factory_class</b> to <b>org.hibernate.transaction.CMTTransactionFactory</b> instead. At first I was a bit sceptical, as CMT makes be thing about EJB containers. However, if you read the Java doc on CMTTransaction it does make sense:

<pre class="brush: java;">
/**
 * Implements a transaction strategy for Container Managed Transaction (CMT) scenarios.  All work is done in
 * the context of the container managed transaction.
 *
 * The term 'CMT' is potentially misleading; the pertinent point simply being that the transactions are being
 * managed by something other than the Hibernate transaction mechanism.
 *
 * Additionally, this strategy does *not* attempt to access or use the {@link javax.transaction.UserTransaction} since
 * in the actual case CMT access to the {@link javax.transaction.UserTransaction} is explicitly disallowed.  Instead
 * we use the JTA {@link javax.transaction.Transaction} object obtained from the {@link TransactionManager}
</pre>

After that everything seems to work fine. So to conclude, if you want hibernate to manage the JTA transaction via the UserTransaction, you should use JtaTransactionFactory. In that case you must use the Hibernate API to control the transaction. If there is someone else managing the transaction (Spring, EJB container ...) you should use CMTTransactionFactory instead. Hibernate will then revert to registering synchronisations, by checking for active    javax.transaction.Transaction using the javax.transaction.TransactionManager. If there is any other issue popping up I'll update this entry accordingly.
