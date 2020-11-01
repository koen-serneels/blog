---
layout: post
title: Oracle AQ with Spring
date: '2011-10-07T15:48:00.000+02:00'
author: Koen Serneels
img: 2.png
tags:
- JMS
modified_time: '2013-08-02T12:35:50.964+02:00'
blogger_id: tag:blogger.com,1999:blog-3406746278542965235.post-7951141864096902361
blogger_orig_url: http://koenserneels.blogspot.com/2011/10/oracle-aq-with-spring.html
---

In my current project we use a Java SEDA. The MOM to support this is IBM (Websphere) MQ. Our most used object is the queue, which enables us to handle events asynchronously and by multiple consumers which greatly improves scalability and robustness.

From a Java point of view the MOM implementation is really not that important, as it is accessed via the JMS API. So whether its Websphere MQ, JBoss MQ, ... as long as it has JMS support its pretty transparent. We do use some minor MQ specific extensions (PCF) to get queue depths and the likes, but that is more from an operational management point of view.

The choice of MQ was made before I joined the project, probably because other legacy subsystems have the least trouble dealing with MQ since they are already IBM based. Although we don't benefit a lot from it possibilities, since there is no QueueManager to QueueManager communication or the likes in which MQ is really strong. But it has to be said that MQ is a solid and mature product with a lot of possibilities.

The downside is probably its price (especially if you under-use it) and that it requires specific MQ knowledge to operate and maintain a running instance. For example; moving messages from a queue to another natively on Solaris is not a trivial thing if your not into the MQ administration (no, the 'MOVE' command is not supported on MQ Solaris).

Since we are using 2 resources most of the time, this also implies that our backends are running XA transactions to make 2PC work between our MOM and RDBMS (Oracle).
A while ago someone threw the idea on the table to switch to Oracle AQ (Advanced Queues) which is the Oracle MOM implementation. I'm not going in the area of comparing MQ vs AQ, but the fact is that AQ supports JMS and is a fully fledged MOM (It also has Topics, btw), so on paper it is more then enough for our usages.

Cool detail is that JMS Connection that you obtain is actually backed by a normal (JDBC) database connection. In fact, what happens is that the AQ driver uses a datasource under the hood.  If you do a Queue.publish() or a Queue.read() the AQ driver will translate that to stored procedure and send them through the SQL datasource you instantiated it with. This also means we could drop our XA, since we only need to enlist a single resource for both our MOM and RDBMS access.

<div class="separator" style="clear: both; text-align: center;">
<a href="{{site.baseurl}}/assets/img/2011-10-07-oracle-aq-with-spring/oracleaq-large.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="558" data-original-width="1090" height="124" src="{{site.baseurl}}/assets/img/2011-10-07-oracle-aq-with-spring/oracleaq-small.png" width="440" /></a></div>

To set this up my first idea was to look for a resource adapter (RAR) which would enable AQ via the application server (Webshere MQ also ships with a JEE RAR). At that point I did not knew how it would handle the JDBC connection sharing if connection would be made via the RAR, but anyway. Quickly I found out that there is no real AQ resource adapter available for other JEE servers then Oracle AS itself (for this I was using Glassfish btw).

There is: <a target="_blank" href="http://genericjmsra.java.net/">genericjmsra</a> but you cannot use it "properties based" like you enter the uri/username/password of the MOM. See <a target="_blank" href="http://genericjmsra.java.net/docs/oracleaq/glassfish-oracleaq.pdf">here</a> for its AQ specific manual:

Quote:
<i>
Oracle JMS client does not allow creation of ConnectionFactory, QueueConnectionFactory or
TopicConnectionFactory utilizing JavaBean approach. The factory creation is only possible through AQjmsFactory class provided in the Oracle jms client api. However fortunately, Oracle does support the JNDI lookup approach. We will be focusing on the JNDI approach for Oracle AQ and glassfish integration
</i>
This means you need an Oracle LDAP server in which some remote objects are published which are then looked up by the RA. So sharing the same JDBC connection for relational access and AQ will certainly not be possible this way.

Fortunately you can use the AQJmsFactory (that's the main factory which you feed a datasource and it gives you back a JMS ConnectionFactory) directly from your code, but that would require some boiler plate code as the AQJmsFactory checks that the actual connection is a direct Oracle connection.

If you are using a JDBC pool, like for example C3PO or Commons DBCP they will wrap the connections (in order to suppress closes etc) and these connections will be rejected because they are no direct instance of the Oracle connection. Thankfully a new Spring module was released at the right time and comes to the rescue with: <a target="_blank" href="http://www.springsource.org/spring-data/jdbc-extensions">Spring jdbc-extensions</a>.

This is that boiler plate you want to seamlessly integrate Oracle AQ with your existing Spring managed datasource and transactions. The extension will make sure the Oracle AQJmsFactory is given a proxy which will be an instance of the Oracle connection. The proxy enables us to control what we give the Oracle AQ implementation. 

For example when it tries to call 'close' we will suppress the call, since we know it will be handled by transaction manager (datasource,hibernate, jta, ...) later on. If your interested in this check the source at: org.springframework.data.jdbc.config.oracle.AqJmsFactoryBeanFactory. 
That is the custom namespace handler for the AQ Spring XML config which creates the appropriate beans to do the boiler plate.

In this first example we create a scenario in which an event is received (Q1), a database record is inserted (T1) and a second event is published (Q2). All of this should run in one transaction, so if there is a failure at any point everything should be reverted (1 message back on Q1, no records in T1, and no messages on Q2). If everything succeeds, the message from Q1 should be processed, the record inserted and a new message published on Q2.

To start I'm going to setup the two AQ queue's and their queue table:

<pre class="brush: sql;">
EXECUTE DBMS_AQADM.CREATE_QUEUE_TABLE(queue_table => 'Q1_T', queue_payload_type => 'SYS.AQ$_JMS_TEXT_MESSAGE');
EXECUTE DBMS_AQADM.CREATE_QUEUE (Queue_name => 'Q1',  Queue_table => 'Q1_T', max_retries => 2147483647);
EXECUTE DBMS_AQADM.START_QUEUE (Queue_name => 'Q1');

EXECUTE DBMS_AQADM.CREATE_QUEUE_TABLE(queue_table => 'Q2_T', queue_payload_type => 'SYS.AQ$_JMS_TEXT_MESSAGE');
EXECUTE DBMS_AQADM.CREATE_QUEUE (Queue_name => 'Q2',  Queue_table => 'Q2_T', max_retries => 2147483647);
EXECUTE DBMS_AQADM.START_QUEUE (Queue_name => 'Q2');
</pre>

On AQ each Queue needs to have a corresponding queue table. The queue table is the table where the data is physically stored. You will never talk to a queue table directly, but you can use it with DML to query them via your favorite database IDE. On each you can specifiy additional properties, on the queue table you have to specifiy which payload it will have. On the queue itself you can specifiy after how many unsuccesful dequeues the message is moved to the exception queue. 

In our project we make use of an application level failover and DLQ management system with separate queueing. So we don't need this feature. There is however no way to turn this off, so we've chosen the max setting (which is Integer.MAX_VALUE). Btw; the exception queues are generated automatically, you have no control over them.

To check if everything is created:

<pre class="brush: sql;">
select * from all_queues where name like 'Q1%' or name like 'AQ$_Q1%' or name like 'Q2%' or name like 'AQ$_Q2%'
</pre>

The results:

<div align="center"><table border="1"  cellpadding="0" cellspacing="0"><tr>   <th>NAME</th>  <th>QUEUE_TABLE</th>  <th>QID</th>  <th>QUEUE_TYPE</th>  <th>MAX_RETRIES</th> </tr><tbody id="data"><tr> <td>Q2</td> <td>Q2_T</td> <td align="right">365831</td> <td>NORMAL_QUEUE</td> <td align="right">2147483647</td>   </tr><tr> <td>AQ$_Q2_T_E</td> <td>Q2_T</td> <td align="right">365830</td> <td>EXCEPTION_QUEUE</td> <td align="right">0</td>   </tr><tr> <td>Q1</td> <td>Q1_T</td> <td align="right">365816</td> <td>NORMAL_QUEUE</td> <td align="right">2147483647</td>   </tr><tr> <td>AQ$_Q1_T_E</td> <td>Q1_T</td> <td align="right">365815</td> <td>EXCEPTION_QUEUE</td> <td align="right">0</td>   </tr></tbody></table></div>
Next we'll setup our Spring config. The goal is to create a message consumer that listens for messages on Q1 and processes them. Our processing will consist of inserting a record in T1 and putting a message on Q2.

<pre class="brush: xml; highlight: [23, 37];">
&lt;!-- Sets up the JMS ConnectionFactory, in this case backed by Oracle AQ --&gt;
 &lt;bean id=&quot;oracleNativeJdbcExtractor&quot; class=&quot;org.springframework.jdbc.support.nativejdbc.SimpleNativeJdbcExtractor&quot;/&gt;
 &lt;orcl:aq-jms-connection-factory id=&quot;connectionFactory&quot; data-source=&quot;dataSource&quot; use-local-data-source-transaction=&quot;true&quot; native-jdbc-extractor=&quot;oracleNativeJdbcExtractor&quot;/&gt;

 &lt;tx:annotation-driven/&gt;

 &lt;bean id=&quot;dataSource&quot; class=&quot;org.apache.commons.dbcp.BasicDataSource&quot; lazy-init=&quot;true&quot;&gt;
  &lt;property name=&quot;driverClassName&quot; value=&quot;oracle.jdbc.driver.OracleDriver&quot;/&gt;
  &lt;property name=&quot;url&quot; value=&quot;jdbc:oracle:thin:host:port:SID&quot;/&gt;
  &lt;property name=&quot;username&quot; value=&quot;Scott&quot;/&gt;
  &lt;property name=&quot;password&quot; value=&quot;Tiger&quot;/&gt;
 &lt;/bean&gt;

 &lt;!-- Using DataSourceTxManager, but could also be HibernateTxManager or JtaTxManager --&gt;
 &lt;bean id=&quot;transactionManager&quot; class=&quot;org.springframework.jdbc.datasource.DataSourceTransactionManager&quot; lazy-init=&quot;true&quot;&gt;
  &lt;property name=&quot;dataSource&quot; ref=&quot;dataSource&quot;/&gt;
 &lt;/bean&gt;

 &lt;!-- You can also construct the JMSTemplate in code, but we'll do it here so its all together in one place --&gt;
 &lt;bean id=&quot;jmsTemplate&quot; class=&quot;org.springframework.jms.core.JmsTemplate&quot;&gt;
  &lt;property name=&quot;connectionFactory&quot; ref=&quot;connectionFactory&quot;/&gt;
  &lt;property name=&quot;defaultDestinationName&quot; value=&quot;Q2&quot;/&gt;
  &lt;property name=&quot;sessionTransacted&quot; value=&quot;true&quot;/&gt;
 &lt;/bean&gt;

 &lt;bean id=&quot;myMessageListener&quot; class=&quot;be.error.jms.MyMessageListener&quot;&gt;
  &lt;property name=&quot;dataSource&quot; ref=&quot;dataSource&quot;/&gt;
  &lt;property name=&quot;jmsTemplate&quot; ref=&quot;jmsTemplate&quot;/&gt;
 &lt;/bean&gt;

 &lt;!-- Once it is started, it will try to read messages from Q1 and let 'messageListener' process them --&gt;
 &lt;bean id=&quot;messageListenerContainer&quot;class=&quot;org.springframework.jms.listener.DefaultMessageListenerContainer&quot;&gt;
  &lt;property name=&quot;connectionFactory&quot; ref=&quot;connectionFactory&quot;/&gt;
  &lt;property name=&quot;transactionManager&quot; ref=&quot;transactionManager&quot;/&gt;
  &lt;property name=&quot;destinationName&quot; value=&quot;Q1&quot;/&gt;
  &lt;property name=&quot;messageListener&quot; ref=&quot;myMessageListener&quot;/&gt;
  &lt;property name=&quot;sessionTransacted&quot; value=&quot;true&quot;/&gt;
 &lt;/bean&gt;
</pre>As you can see the magic is in the orcl:aq-jms-connection-factory which will make a JmsConnectionFactory available under the id 'connectionFactory' and using our datasource to do the AQ queueing.

<b>Very important:</b> if you don't want to spend half a day investigating some weird transaction behaviour (I even mistakenly thought it was a bug and pointed that out <a target="_blank" href="https://jira.springsource.org/browse/DATAJDBC-9">here</a>) I suggest to read this: 

In my configuration you will see that the 'sessionTransacted' is set to "true" for the JmsTemplate and for the DefaultMessageListenerContainer. This makes sense as we are running outside of a JEE managed environment and we want to have local transactions for our JMS operations. The theory behind it is however a bit more complex.

When running outside of a JEE managed environment you have the choice of letting your session interaction be part of a local transaction. This is controlled by the sessionTransacted setting (it maps directly on the JMS API). This means that if you consume messages from different objects belonging to the same session, they will be controlled in a single transaction.
For example, I create QueueSession #1 and I use it to consume a message from Q1 and consume another message from Q2. After consuming both messages, I can issue a session.rollback() and everything is brought back to its initial state. If I would have used no transactions, I would be working with an acknowledgement mode. Suppose I would have chosen CLIENT_ACKNOWLEDGE then I had to acknowledge on message level whether my message was successfully consumed. So I would have first retrieved message #1 from Q1 and then message #2 to Q2 (all via QueueSession #1). In the end I would have to do:

<pre class="brush: java;">messageOne.acknowledge();
//system crashes here
messageTwo.acknowledge();
</pre>

This could of course create inconsistency as in my example messageOne was marked consumed but messageTwo wasn't. This is only a problem if your unit of work should be treated in an atomic way. If it is you should use at least local transactions.

When you want to consume/produce messages from a Queue and do interaction with another resource (RDBMS) for example you should use a distributed transactionmanager (in our case that would mean JTA). But remember that we are not dealing with different resources here, it all comes down to a single database connection. So in our case the "local transaction" is a bit "longer local" then it would normally be as it also includes all our (SQL) calls made to that same database connection as the JMS infrastructure is using.

In our case the DataSourceTransactionManager will control the local transaction, and that includes JMS operations as well as SQL operations issued via JDBC. It is that component which will call commit or rollback. there is no need for intermediate commits on the queueSession. 

So basically: by setting sessionTransacted to true, no one performs intermediate commits and leaves everything to whoever controls the transaction, in our case DataSourceTransactionManager.
Make sure you use JdbcTemplate for direct JDBC access and JmsTemplate for MOM access. Make sure sessionTransacted is set to true when you should create JmsTemplate in code. Also, the DefaultMessageListenerContainer is a JMS receiver and must also be sessionTransacted for the same reason.

You might want to be tempted to remove the sessionTransacted from the JmsTemplate and DefaultMessageListenerContainer if you are running in an JEE environment. The JMS API says that the values to sessionTransacted and acknowledgementMode are ignored in such case.
While this is true in general, it is not true in this case. The Oracle AQ will not properly detected that it is running in a JEE JTA environment if you are using anything else then Oracle AS. If you remove the property, then the driver will perform intermediate commits and your transaction will be broken. So also in JEE mode you will have to leave this set to true!

But don't worry, in the JTA case your datasource will then be XA enabled and the transactionmaanger performing commmits will be the JtaTranasctionManager. As far as the AQ driver is concerned it sees no difference (all transaction handling an coordination is done at an higher level).

Also, I'm using a DataSourceTransactionManager here, since I only require direct JDBC access.
If you would be using hibernate, you could use HibernateTransactionManager. You could then do AQ, plain JDBC access and work with hibernate's SessionFactory at the same time.
If you would still have another resource (maybe a 2nd RDBMS) and still want XA, you can simply plugin the JTA transaction manager without any problem (its just a matter of switching configuration).

For the Java messageListener part, this is all standard:

<pre class="brush: java;">public class MyMessageListener implements SessionAwareMessageListener&lt;Message&gt; {

 private DataSource dataSource;
 private JmsTemplate jmsTemplate;


 @Override
 public void onMessage(Message message, Session session) throws JMSException {
  //Message received from Q1 via 'messageListenerContainer'
  TextMessage textMessage = (TextMessage) message;
  System.out.println("Received message with content:" + textMessage.getText());

  //Insert its content into T1
  new JdbcTemplate(dataSource).update("insert into T1 values (?)", textMessage.getText());
  System.out.println("Inserted into table T1");

  //Publish a message to Q2
  jmsTemplate.send(new MessageCreator() {
   @Override
   public Message createMessage(Session session) throws JMSException {
    TextMessage textMessage = session.createTextMessage();
    textMessage.setText("<somexml>payload</somexml>");
    return textMessage;
   }
  });
  System.out.println("Sended message to Q2");
 }

 public void setDataSource(DataSource dataSource) {
  this.dataSource = dataSource;
 }

 public void setJmsTemplate(JmsTemplate jmsTemplate) {
  this.jmsTemplate = jmsTemplate;
 }
}
</pre>

I then created a small forever blocking test case to quickly fire up the application context so that the DefaultMessageListenerContainer could start looking for messages on Q1. 

<pre class="brush: java;">
@Test
@ContextConfiguration(locations = { "classpath:/spring/aq-test.xml" })
public class OracleAqTransactionResourceTest extends AbstractTestNGSpringContextTests {

 @Autowired
 private DataSource dataSource;
 private JdbcTemplate jdbcTemplate;

 @BeforeMethod
 public void setup() {
  jdbcTemplate = new JdbcTemplate(dataSource);
 }

 public void testSingleTransaction() {
  System.out.println("Running...");
  blockUntillReadyOrTimeout();
  System.out.println("Done.");
 }

 private void blockUntillReadyOrTimeout() {
  while (true) {
   try {
    Thread.sleep(2000);
   } catch (InterruptedException e) {
    throw new RuntimeException(e);
   }
  }
 }
}
</pre>

After launching the test, I inject a message into Q1 (I use Oracle SQL developer):

<pre class="brush: sql;">
DECLARE
    msg SYS.AQ$_JMS_TEXT_MESSAGE;
    queue_options       DBMS_AQ.ENQUEUE_OPTIONS_T;
    message_properties  DBMS_AQ.MESSAGE_PROPERTIES_T;
    message_id RAW(30);

BEGIN
      msg := SYS.AQ$_JMS_TEXT_MESSAGE.CONSTRUCT();  
      msg.set_text('testing 123');
      DBMS_AQ.ENQUEUE(
        queue_name => 'Q1',
        enqueue_options => queue_options,
        message_properties => message_properties,
        payload => msg,
        msgid => message_id);
        commit;
END;
</pre>

And off we go:

<pre>Running...
Received message with content:testing 123
Inserted into table T1
Sended message to Q2
</pre>

In oracle we see that the message is present on Q2 (at least its queue table):

<div class="separator" style="clear: both; text-align: center;">
<a href="{{site.baseurl}}/assets/img/2011-10-07-oracle-aq-with-spring/sqldev1-large.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="558" data-original-width="1090" height="124" src="{{site.baseurl}}/assets/img/2011-10-07-oracle-aq-with-spring/sqldev1-small.png" width="440" /></a></div>

inserted into T1:

<div class="separator" style="clear: both; text-align: center;">
<a href="{{site.baseurl}}/assets/img/2011-10-07-oracle-aq-with-spring/sqldev2-large.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="558" data-original-width="1090" height="124" src="{{site.baseurl}}/assets/img/2011-10-07-oracle-aq-with-spring/sqldev2-small.png" width="440" /></a></div>

You are free to play with some transaction scenario's, as creating multiple (possibly nested) transactions, let them rollback etc. I performed 5 scenario's and they all worked fine.

PS. make sure you use at least spring-jdbc 1.0_M2 (or up) since we discovered a <a target="_blank" href="https://jira.springsource.org/browse/DATAJDBC-8">small bug</a> in M1 which could cost you some time to investigate :)
