---
layout: post
title: java.text.Collator
date: '2011-03-26T03:21:00.000+01:00'
author: Koen Serneels
img: 2.png
tags:
- Java SE
modified_time: '2011-03-26T16:38:24.708+01:00'
category: java
blogger_orig_url: http://koenserneels.blogspot.com/2011/03/javatextcollator.html
---

Some time ago I got introduced to a part of the Java text API that was unexplored territory for me: the <a target="_blank" href="http://download.oracle.com/javase/6/docs/api/java/text/Collator.html">Collator</a>. Languages imply more complexity then one on first sight would think (check <a target="_blank" href="http://www.unicode.org/versions/Unicode6.0.0/UnicodeStandard-6.0.pdf">this</a> if you have any doubt).

The main usage of the Collator is to help us with a part of that linguistic complexity, more specifically locale sensitive comparison. It implements the collation specification defined by the <a target="_blank" href="http://unicode.org/reports/tr10/">Unicode</a>

The Java Collator roughly does these things:

<ul>
	<li>Canonicalization of canonical equivalent characters</li>
	<li>Multi level comparison</li>
</ul>

Comparing Java based Strings works by comparing their Unicode code point that maps with the character. This would mean that the position of the character in the Unicode code charts specifies the sorting weight, but that is not the case. Languages might have different sorting weights for the exact same characters.

For example, if you don't know anything about the German language, you might expect that ß (\u00DF) is sorted as it was a 'b' or 'B'. That is not correct, since its actually represents the combination 'ss'. But even knowing this, a standard comparison with ß would yield false results since its code point is higher then a normal 's'. So in the end it will not be sorted as an 'ss' but it will be sorted as it was 'higher' then 'z'.

Multi level comparison solves this by offering 4 comparison levels: base letters, accents, case, punctuations. If the first level is used, only base character differences are considered. With the second level base characters as well as accents are considered significant, etc

Note: the Collator apparently does not support punctuation.

<pre class="brush: java;">
System.out.println("a equals b -> " + (collator.compare("a", "b")==0 ? "true":"false"));
System.out.println("a equals à -> " + (collator.compare("a", "à")==0 ? "true":"false"));
System.out.println("A equals a -> " + (collator.compare("a", "A")==0 ? "true":"false"));
</pre>

With collator.setStrength(Collator.PRIMARY):

<pre class="brush: java;">
a equals b -> false
a equals à -> true
A equals a -> true
</pre>

With collator.setStrength(Collator.SECONDARY);

<pre class="brush: java;">
a equals b -> false
a equals à -> false
A equals a -> true
</pre>

With collator.setStrength(Collator.TERTIARY);

<pre class="brush: java;">
a equals b -> false
a equals à -> false
A equals a -> false
</pre>

Our first use case for which we used the Collator was for the first function; canonicalization. Unicode foresees different ways of representing certain characters. For example; ü is identified by a single code point \u00FC and thus a single character. However, it is also possible to form ü with a character + diacritical mark(<a href="#note1">°</a>): u (\u0075) and ¨ (\u0308).

It makes sense if you think about it, on a classic typewriter you would also form ü by first printing u, go back one position and then print ¨. The ¨ is a so called invisible character on the type writer. On our keyboard we can do the same. You can press the button marked ü or you can: &lt;altgr&gt; + &lt;¨&gt; + &lt;u&gt; which gives you ü.

The end result (on your screen at least) is the same disregarding how you form ü, however, it is stored differently. The single character ü would be saved as: 0x00FC. If you would have formed ü by typing ¨ followed by u, it would be saved as: 0x0075 0x0308

As long as you just want print those characters, in a browser, console, text editor, ... you can (hopefully) rely on that software to display it right. However, if you are writing Java and want to do operations with character streams containing such characters it becomes tricky.

Example (<a href="#note2">°°</a>): lets take a typical German word such as "abgaskrümmerdichtung":

<pre class="brush: java;">
String single = "abgaskr\u00FCmmerdichtung";
String combined = "abgaskr\u0075\u0308mmerdichtung";

System.out.println("Single equals combined? " + single.equals(combined));
System.out.println("Single: " + single);
System.out.println("Combined: " + combined);
</pre>

The first line will say that they are not equal: Single equals combined? false<br />However, when both are displayed, they look exactly the same:<br /><br />Single: abgaskrümmerdichtung<br />Combined: abgaskrümmerdichtung

Our software needs comparison on a higher level rather then pure code point comparison. It needs to be canonicalized first, by something that knows that \u0075\u0308 is in fact \u00FC. Collator to the rescue:

<pre class="brush: java;">
collator.setDecomposition(Collator.CANONICAL_DECOMPOSITION);
String single = "abgaskr\u00FCmmerdichtung";
String combined = "abgaskr\u0075\u0308mmerdichtung";

System.out.println("Single equals combined? " + (collator.compare(single, combined) == 0 ? "true": "false"));
</pre>

This will print; Single equals combined? true

A second use-case where the Collator came in handy: we were in need to map characters from ISO8859-1 to 7bit ASCII. ISO8859-1 contains several accented characters that do not exist in 7bit ASCII. Our goal is to map these characters to their canonical equivalent that is supported in 7bit ASCII. For example: "çéàëê" could be mapped to "ceaee". Of course, other characters for which no obvious equivalence exist cannot me mapped (and will be converted as '?')

Remember: Java uses Unicode and UTF16 as encoding. ASCII and the ISO8859 family are both character maps and encodings in one. Unicode and ISO8859-1 share the same code points for the first 256 glyphs. They are also compatible on encoding level: if you take the letter 'A' and save it (codepoint + encoding = ISO8859) and you decode it as Unicode/UTF8, it will still print 'A'. (this is not the case with UTF16/32). ASCII only shares the first 128 glyphs (extended, 8bit, ASCII is not compatible with Unicode or ISO8859-1).

<pre class="brush: java;">
String ascii7Characters = &quot; !\&quot;#$%&'()*+,-./0123456789:;&lt;=&gt;?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\\]^_`abcdefghijklmnopqrstuvwxyz{|}~&quot;;
Collator ascii7Collator = Collator.getInstance(new Locale(&quot;nl&quot;, &quot;BE&quot;));
ascii7Collator.setStrength(Collator.PRIMARY);

Map&lt;CollationKey, Character&gt; ascii7CollationMappings = new HashMap&lt;CollationKey, Character&gt;();

for (char c : ascii7Characters.toCharArray()) {
ascii7CollationMappings.put(ascii7Collator.getCollationKey(String.valueOf(c)), c);
}

String accented = &quot;çéàëê&quot;;
StringBuilder ascii7 = new StringBuilder();

for (Character character : accented.toCharArray()) {
Character canonicalizedCharacter = ascii7CollationMappings.get(ascii7Collator.getCollationKey(String.valueOf(character)));
ascii7.append(Character.isUpperCase(character) ? Character.toUpperCase(canonicalizedCharacter): canonicalizedCharacter);
}

System.out.println(&quot;ISO8859-1 converted to 7bit ASCII:&quot; + ascii7.toString());
</pre>

This will print: ISO8859-1 converted to 7bit ASCII:ceaee

The String created on line 12 is of course Unicode (and encoded in UTF16, but not relevant now).
But since these characters are in the range which is equal between ISO8859-1 and Unicode, it actually does not matter. In real life we would be reading in a byte stream which is explicitly decoded as ISO8859-1:

<pre class="brush: java;">
String accented = new String(inputInIso8859_1, "ISO8859-1");
</pre>

What we did here is use the collator its <a target="_blank" href="http://download.oracle.com/javase/6/docs/api/java/text/CollationKey.html">CollationKey</a> and bind it to our normalized 7bit ASCII character. These key are the canonicalized form of the character, depending on the strength and decomposition values you configured the Collator with. Characters that are canonical equal will also have the same CollationKey. You can use the CollactionKey for linguistically correct sorting/searching, since it will yield the correct order based upon the  <a target="_blank" href="http://download.oracle.com/javase/6/docs/api/java/util/Locale.html">Locale</a> you initialized the Collator with (it implements Comparable).

<span style="font-size:80%"><a name="note1">(°)</a>Diacritical marks are special glyphs, in that way that they are combined with other glyphs and say something about the intonation. For example: <`> can be considered a diacritical mark.
</span>

<span style="font-size:80%"><a name="note2">(°°)</a>The so called code points named in this text refer to glyphs in the Unicode map. They are shown in Java escaped hexadecimal form notation, so \u + 16bit hex. The encoded form is UTF16, with the given examples this means that it is 16bit per character and the code point matches with the encoded form (since the code points in the examples are between \u0000...\uD7FF and \uE000...\uFFFF)</span>
