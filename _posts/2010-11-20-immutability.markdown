---
layout: post
title: Immutability
date: '2010-11-20T03:40:00.000+01:00'
author: Koen Serneels
img: 1.png
tags:
- Methodology
modified_time: '2011-02-09T13:03:06.435+01:00'
blogger_id: tag:blogger.com,1999:blog-3406746278542965235.post-7049430847124467727
category: design
blogger_orig_url: http://koenserneels.blogspot.com/2010/11/immutability.html
---

Java Puzzlers, the last presentation thursday at Devoxx, came with some new, or at least some refurbished, puzzlers to melt our minds. One however did melt something away in my mind. It had to do with object immutability.  

The puzzler goes like this: Supose you have map, lets say an EnumMap or ConcurrentHashMap. You put some values in that map, and next you obtain the entry set. Now, instead of just doing something with the entries, you add this set to another Collection of type Set using the addAll() method (or constructor, which in fact calls addAll for you). In code this might look like this:

<pre class="brush: java;">
Map&lt;String, String&gt; map = new ConcurrentHashMap&lt;String, String&gt;(); 
map.put("one", "two"); 
map.put("two", "one"); 
Set&lt;Entry&lt;String, String&gt;&gt; set = new HashSet&lt;Entry&lt;String, String&gt;&gt;(map.entrySet()); 
System.err.println(set.size());
</pre>

<sub>Note: that the actual presented puzzler was in a more  entertaining form</sub>

You'll notice that the set.size() will return 1, and not 2. The reason behind is: lack of clarity in the API and an attempted optimization by the implementation. The API does not say anything specific about the mutability of the Entry objects returned in the Set.
Some Map implemenations made the choice of a 'singleton' Entry object instead of creating new immutable Entry objects for each value in the map.

What actually happens is that they change the content of the one and only created Entry for each map value, by calling setters on that sole Entry object. So if a map has 1000 elements, only one Entry object will exist. When the map is dumped to a set of Entries, the map will have called 1000x setKey and setValue on that single Entry object for each element in the map. Funny side effect that if you change the values in the map to something like this:

<pre class="brush: java;">
map.put("keyOne", "valueOne"); 
map.put("keyTwo", "valueTwo");
</pre>

The set.size() DOES print 2 :-) What happens is that the map implementaton uses this construct to detect if an entry already exists:

<pre class="brush: java;">
if (e.hash == hash && ((k = e.key) == key || key.equals(k)))
</pre>

So, the hashCode must be the same AND they must be equal, either through reference or object equality. In the first case, the hashes were equal and the reference of key was also the same. The hashCode on an Entry object is calculated by taking the hashCode of the key exorred with the hashCode of the value. In the first case the hashCode of 'one' exorred with the hashCode of 'two' is the same as the hashCode of 'two' exorred with the hashCode of 'one'. 
Allthough they are distinct Entries (there will not be object equal) the hashCode is the same which is also allowed by the hashCode contract (remember that two objects that are equal must produce the same hashCode, but two non equal objects are not required to produce different hashCodes).

For the second case, the hash actually differs, so they don't bother looking further and the entry is considered as 'distinct'. Of course if you would iterate the set, you would find that the key and value of the Entry object are twice the same. Its unlikley that you run into this problem in your code base, but if you do,  it will guarantee long cosy hours of debug. The reasons why I find this particular 'bug' so interesting are its diverse lessons that we can learn from it:

1. Premature micro optimization comes at a cost. You could argue any optimization is good at this level . Maybe, but I know for sure that for us as application developers we should not attempt such optimizations of our own. You risk creating pitfalls beyond your knowledge while (als in most cases) the gains by such optimizations are very hard to prove
2. Immutability plays an important role in your code. It can avoid 'bugs' such as this one. It gives you a signature when handling an object that it is guaranteerd to be in the same state as it was created with
3. Basic design and code rules apply for everyone. It also shows that it can be forgotten in any project. In this case both the API designer as well as the people who implemented it did not question this
