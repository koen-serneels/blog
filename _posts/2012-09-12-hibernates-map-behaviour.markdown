---
layout: post
title: Hibernate's Map behavior
date: '2012-09-12T16:46:00.001+02:00'
author: Koen Serneels
img: 2.png
tags:
- Hibernate
modified_time: '2012-09-12T21:05:45.753+02:00'
blogger_id: tag:blogger.com,1999:blog-3406746278542965235.post-2077195046369449332
blogger_orig_url: http://koenserneels.blogspot.com/2012/09/hibernates-map-behaviour.html
---

Hibernate has a map construct which makes mapping key/value pairs rather elegant. Let's take a &quot;car&quot; as our example domain object. In our imagination we could store the options of a car as key/value pairs. The options could be things like: engine type, color and so forth. 

The hibernate annotations that we would need to map this car entity are pretty straightforward:  
<pre class="brush: java;">
@Entity
public class Car {

 @Id
 @GeneratedValue(strategy = GenerationType.AUTO)
 private Long id;

 @ElementCollection
 @CollectionTable(name = &quot;options_map&quot;, joinColumns = @JoinColumn(name = &quot;car_id&quot;))
 private Map&lt;String, String&gt; options = new HashMap&lt;String, String&gt;();

 public Long getId() {
  return id;
 }

 public Map&lt;String, String&gt; getOptions() {
  return options;
 }
}
</pre>

We created a &quot;car&quot; entity (and table) having a one-to-many relationship to our map storing the options as key/value pairs in map backed by a options_map table. Now, what will happen with entries having null as value?

You can argue that storing an entry with a null value does not make much sense in most scenarios. This is probably true as the only way you would do something like this is to keep an exhaustive list of possible key values.  Even so, it would be a not so good solution since you would be duplicating your entire key set for each and every car. A better solution would be to map the key values to enumeration instead. Or, if the set of keys are also needed outside of your application, store them in a separate table for example.

Leaving this discussion behind, the thing I want to point out here is that whichever reason you have to store entries with null as their values, think again: hibernate ignores such entries completely.

Lets have a look at how hibernate handles this. In the first example we will create and store a car with three options; engine type, color and cupholder.

<pre class="brush: java;">
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = { InfrastructureContextConfiguration.class, TestDataContextConfiguration.class })
@Transactional
public class CarTest {

 @PersistenceContext
 private EntityManager entityManager;

 @Autowired
 private DataSource dataSource;

 @Test
 public void testWithNonNullValues() throws Exception {
  // Save 3 options with non-null value
  Car merc = new Car();
  merc.getOptions().put(&quot;AIRCO&quot;, &quot;DUAL-AUTO&quot;);
  merc.getOptions().put(&quot;GEARBOX&quot;, &quot;AUTO&quot;);
  merc.getOptions().put(&quot;CUPHOLDER&quot;, &quot;YES&quot;);

  entityManager.persist(merc);
  entityManager.flush();
  entityManager.clear();

  merc = entityManager.find(Car.class, merc.getId());
  Assert.assertEquals(3, merc.getOptions().size());
 }
}
</pre> 

Nothing special happens when we run this test, the test passes: all ok.

Next, we choose to drop an option (the cupholder), but we want to keep all keys existent for each car. So we will again have 3 keys, but the cupholder will have a null value: 

<pre class="brush: java; highlight: [6, 13, 14]">
 public void testWithNullValue() throws Exception {
  // Save 3 properties, 2 with non-null value, 1 with null value
  Car bmw = new Car();
  bmw.getOptions().put(&quot;AIRCO&quot;, &quot;DUAL-AUTO&quot;);
  bmw.getOptions().put(&quot;GEARBOX&quot;, &quot;AUTO&quot;);
  bmw.getOptions().put(&quot;CUPHOLDER&quot;, null);

  entityManager.persist(bmw);
  entityManager.flush();
  entityManager.clear();

  bmw = entityManager.find(Car.class, bmw.getId());
  // FAIL: java.lang.AssertionError: expected:<2> but was:<3>
  Assert.assertEquals(3, bmw.getOptions().size());
 }
</pre>

The test will fail (java.lang.AssertionError: expected:<2> but was:<3>). When we load our entity in an empty session it will only contain 2 options instead of 3. The cupholder option was removed (or simply not stored in database).

This can be a problem; during the lifetime of your transaction the map will continue to have a key 'CUPHOLDER' with a null value. Only after reading data from database again (which will for example happen in a new transaction) the car loaded from database will only have two properties left, since only the properties with a non-null value were stored.
Btw; in our test case we simulate a 'new transaction' by simply flushing and clearing the session; this forces hibernate to load the data freshly again from database as would be the behaviour in a new transaction.

Finally, hibernate also ignores existing records with a null value already in the database (which could be inserted by other users). In the example below we save two options (with non-null value) and then we sneakily add another option with a null value using a plain JDBC insert (we also perform a count directly afterwards to make 100% sure there are effectively three options in the database).

<pre class="brush: java;">
 public void testWithNullValueInsertedByOtherUser() throws Exception {
  // Save 2 options with non-null value
  Car audi = new Car();
  audi.getOptions().put(&quot;AIRCO&quot;, &quot;DUAL-AUTO&quot;);
  audi.getOptions().put(&quot;GEARBOX&quot;, &quot;AUTO&quot;);

  entityManager.persist(audi);
  entityManager.flush();
  entityManager.clear();

  JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
  // Add another option with null value bypassing hibernate (simulating other user)
  jdbcTemplate.update(&quot;insert into options_map (car_id, options_key, options) values (?, 'CUPHOLDER', null)&quot;,
    audi.getId());
  // Verify to make sure we now have 3 options in database
  Assert.assertEquals(3,
    jdbcTemplate.queryForInt(&quot;select count(*) from options_map where car_id = ?&quot;, audi.getId()));

  audi = entityManager.find(Car.class, audi.getId());
  // FAIL: java.lang.AssertionError: expected:<2> but was:<3>
  Assert.assertEquals(3, audi.getOptions().size());

 }
</pre>

The test will fail again: even though our car has 3 options in the database, hibernate will only retrieve the options with a non-null value. So instead of our expected 3 options we only get 2 options in our options map.

Conclusion: during the transaction the domain model will fully reflect the data we stored in it.
If we test for the key 'CUPHOLDER' to be present it would return true.  However, in subsequent transactions, which might read the car and its options we saved before, we will only discover two options since an option with null value is not saved by hibernate and completely ignored. This would render our operation non-idempotent if we really depended on the cupholder key to be present.

To take this a bit further, if you are using Hibernate in combination with an Oracle databases you also need to be careful with empty string values. As you are probably aware Oracle transforms an empty string (a String of length equal to 0) to a null value. So if you would have a column of type varchar2 for example, inserting '' as value will be transformed to null.
So from the Java point of view, when using Oracle, you will also &quot;lose&quot; properties which do not only have a null value but also have a blank string value. 

It is wise to think your null/empty string strategy well through, certainly when using Hibernate in combination with the map construct. The safest solution is probably to not depend on any entry with a blank or null value at all when using this construct. 
