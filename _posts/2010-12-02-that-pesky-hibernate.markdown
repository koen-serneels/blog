---
layout: post
title: That pesky Hibernate
date: '2010-12-02T12:20:00.000+01:00'
author: Koen Serneels
img: 2.png
tags:
- Hibernate
modified_time: '2011-02-08T13:38:07.892+01:00'
blogger_id: tag:blogger.com,1999:blog-3406746278542965235.post-8929328635280149893
category: hibernate
blogger_orig_url: http://koenserneels.blogspot.com/2010/12/that-pesky-hibernate.html
---

Yesterday a collegue of mine asked if I could build some functionality to physically delete records in our database.
Since deleting things is always a good idea, I immediately started to work on his request. As we are using an ORM (Hibernate) to manage our relational persistence, this would just be a matter of adding a single line of code:

<pre class="brush: java;">
session.delete(someEntity);
</pre>

And all the Hibernate magic combined with some cascading would take care of the deletion.So I concluded my assignment fast, and got the well know warm cosy feeling: 'job wel done'.However, suddenly I realised that the combinaton: "hibernate, magic, job well done, warm cosy feeling" has put me into problems before. In fact, I remeber this since last time I was in that position I ended up feeling exactly like <a target="_blank" href="http://www.theluxuryspot.com/wp-content/uploads/2010/03/Scarface-Al-Pacino_l.jpg">this</a> man did.

Just to be sure I went back to the code and configured my logging to print out what Hibernate was doing. As it turned out, something bad was going on. It seems that also Hibernate has difficulties deleting relationships (thats a line to think about). The problem is that each child is deleted one by one. Lets say you have two entities: Person and Product. There is a one to many relationship between Person and Product. The relationship is uni-directional from Person to Product, mapped as a Set, managed by Person (as there is only one side) and the cascading is set to all-delete-orphan.

<pre class="brush: xml;">
&lt;hibernate-mapping default-access="field" &gt;
   &lt;class name="entities.Person" table="person"&gt;
      &lt;id name="id" column="id" access="property"&gt;
         &lt;generator class="native"/&gt;
      &lt;/id&gt;

      &lt;set name="products" cascade="all-delete-orphan"&gt;
         &lt;key column="person_id" not-null="true" update="false" foreign-key="person_fk"/&gt;
         &lt;one-to-many class="entities.Product"/&gt;
      &lt;/set&gt;
   &lt;/class&gt;
&lt;/hibernate-mapping&gt;
</pre>

<pre class="brush: xml;">&lt;hibernate-mapping default-access="field" &gt;
   &lt;class name="entities.Product" table="product"&gt;
   
      &lt;id name="id" column="id" access="property"&gt;
         &lt;generator class="native"/&gt;
      &lt;/id&gt;
      
      &lt;property name="name" column="name"/&gt;
   &lt;/class&gt;
&lt;/hibernate-mapping&gt;
</pre>

If you delete a Person object which has 5 products, you will see that pesky Hibernate doing this:

<pre>31398 [main] DEBUG org.hibernate.SQL  - delete from product where id=?31399 
[main] DEBUG org.hibernate.SQL  - delete from product where id=?31399 
[main] DEBUG org.hibernate.SQL  - delete from product where id=?31399 
[main] DEBUG org.hibernate.SQL  - delete from product where id=?31399 
[main] DEBUG org.hibernate.SQL  - delete from product where id=?31400 
[main] DEBUG org.hibernate.SQL  - delete from person where id=?
</pre>

In some conditions this might be acceptable, however, in our case there could be thousands of childs records.I do know that with bulk operations and large datasets Hibernate is maybe not the best choice.However, we configured Hibernate that it just worked for our case.For example; in our real scenario the "lazy" attribute on the Set has been set to "Extra"This allows to get a count on the child Collection without Hibernate loading every child.Other bulk operations we managed by doing HQL. So in the end it worked out very good (and fast) so we where happy with the ORM functionality without having to pay performance costs.So I was not planning to let this problem become a performance bottleneck.As it turns out this 'problem' is very known one and there is lots to be found on the internet. I learned that Hibernate should normally be able to do a 'single shot delete'. It is even mentioned in the manual! So I tried every possible combination, but could not force hibernate to do that 'single shot delete'.Secondly I tried to do it the 'bulk' way by just issuing some delete HQL.In our case its not a problem that it bypasses the Session, since the transaction consists only out of the delete.YAP! As it turned out, in my real scenario the Child has a Hibernate &lt;map&gt; relation to a properties table.When doing an HQL delete, the cascading on the &lt;map&gt; is not respected. This seems to be a issue as well (<a target="_blank" href="http://opensource.atlassian.com/projects/hibernate/browse/HHH-695">here</a>).

I also tried to delete the &lt;map&gt; manually using HQL, but that doesn't work since there is no entity to 'grab' in the query. Thirdly, I learned about the "on-delete" attribute which you can set on the &lt;key&gt; element in the &lt;Set&gt;.The only requirement is that the Set must be the non-managed side of the relation (so inverse must be 'true').In my Person/Product example this is not the case (as there is only one side). But in my real scenario, which is slightly different, this is the case: the association is mapped bi-directionally and the Child is owner of the relationship. If you enable the on-delete = "cascade" Hibernate would leave the deletion to the database. In other words, it will depend on the database cascading delete, so your table DDL should have the 'on delete cascade' in its FK constraint DDL. Cool note, that if you use hbm2ddl Hibernate generates the correct DDL including the on delete cascade. 

Good, good! The only thing I now had to do was to tell our db administrator to add the 'on delete cascade'. But wait, I also still have the &lt;map&gt;, So I tried to add the 'on-delete= cascade' there as well. BUT! on-delete = 'cascade' can only be added on inverse associations!The &lt;map&gt; is mapped uni-directional and is therefore always the owner. So, bad luck once again, this did not work. Agreed: at this point I could refactor the &lt;map&gt; construct to a normal stand alone Entity and make a bidrectional one-to-many/many-to-one. But that would imply many hours refactoring for something that was just working fine, so I felt not doing this. Fourthly (and finally) I decided to just bypass Hibernate completely for this delete. I created a named query :

<pre class="brush: xml;">
&lt;sql-query name="deletePerson">delete from person where id = :personId&lt;/sql-query&gt;
</pre>

I asked the database administrators to add 'on delete cascade' to the FK's of Product and the map table. To be able to test this in our in-memory database, I added some extra DDL in the Person mapping:

<pre class="brush: xml;">&lt;database-object&gt;
 &lt;create&gt;
  alter table product drop constraint person_fk
  alter table product add constraint person_fk foreign key (person_id) references person(person_id) on delete cascade
  &lt;!-- same here for the &lt;map&gt; relation--&gt;
 &lt;/create&gt;
 &lt;drop/&gt;
&lt;/database-object&gt;  
</pre>

This allows to simulate the database cascading in my tests (which run against an in-memory database).So to conclude my journey, the steps you can try if you want a fast delete:1. Do not use Hibernate2. Just use the not working one-shot-delete, so you don't have to complain about Hibernate in some blog entryNow as for your real options:1. Map your 'one' side of the association with on-delete="cascade" and alter the FK constraints in the database with cascading2. Use HQL to delete the entities yourself (do not forget this is a bulk operation and the delete is not reflected in the Session!)3. Bypass Hibernate and use plain SQL + database cascading (same remark about the session)Oh, btw, if you use the first suggestion, be aware that (even if you are using lazy true/extra on the association) Hibernate always loads the complete association before issuing the delete. Ain't that cute or what?So if you have large datasets going on and you want performance, afaik you are limited to 2 and 3.
