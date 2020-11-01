---
layout: post
title: 'WS-Security: using BinarySecurityToken for authentication'
date: '2013-09-08T15:26:00.000+02:00'
author: Koen Serneels
tags:
- Web Services
modified_time: '2013-09-09T11:29:28.577+02:00'
blogger_id: tag:blogger.com,1999:blog-3406746278542965235.post-579438896262037031
blogger_orig_url: http://koenserneels.blogspot.com/2013/09/ws-security-using-binarysecuritytoken.html
---

As we all know, one goal set by WS-Security is to enforce integrity and/or confidentially on SOAP messages. In case of integrity, the signature which is added to the SOAP message is the result of a mathematical process involving the private key of the sender resulting in an encrypted message digest.
 Most frameworks, such as WSS4J, will by default only sign the body. If you're adding extra headers, such as a Timestamp header, you'll have indicate explicitly to sign them. Using the Spring support for WSS4J for example, you can set a comma separated list containing the local element name and the corresponding namespace using the <i>securementSignatureParts</i> property.
 Below an example how to instruct it to both sign the Body and Timestamp element (and their siblings). This will result in two digital signatures being appended to the message: 

<pre class="brush: xml;">
&lt;property name="securementSignatureParts" value="{}{http://schemas.xmlsoap.org/soap/envelope/}Body;{}{http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd}Timestamp" /&gt;
</pre>

Eventually the SOAP message will be send together with the XML digital signature data and in most cases a BinarySecurityToken containing the certificate.
Nothing new so far. However, what struck me is that it seems not widely understood what the goal is of the BST neither how authentication is controlled using it. Let me try to shed some light on this:

The certificate of the sender which is send along with the SOAP message plays the role of identification. You can compare it as being the username++. It should be clear that the certificate inside the message cannot be trusted, neither can a username without verifying the password. So far everyone agrees on that: <i>"yeah of course, certificates need to be validated in order to be trusted and then you're set!"</i>

But that is not the entire story. Validation of the certificate is not the same as authentication. The fact that the certificate in the message is valid and is signed by a known CA is not enough to consider the sender authenticated.

For example: I, in my most malicious hour, could have intercepted the message, changed the content, created a new signature based on my private key and replaced the BST in the message with my certificate. My certificate could perfectly be an official CA signed certificate (even signed by the same CA as you're using) so it would pass the validation check. If the framework would simply validate the certificate inside the message we would have no security at all.

 <i>Note: If you're sending the message over secure transport instead, chances are that I was not able to intercept the message. But secure transport is mostly terminated before the actual endpoint, leaving a small piece of the transport "unsecured". Albeit this part will be mostly internally in your company, but what I want to point out is that no matter how secure your transport is, the endpoint has the end responsibility in verifying the identity of the sender. For example; in an asynchronous system the SOAP message could have been placed on a message queue to be processed later.  When processing starts by the endpoint, the trace of the secure transport is long gone. You'll have to verify the identity using the information contained in the message.</i>

In order to close this loophole we have two solutions:

The first solution builds further on what we already described: the certificate in the message is verified against the CA root certificates in the truststore. In this scenario it advised to first narrow the set of trusted CA's. You could for example agree with your clients on a limited list of CA's to get your certificates from. Doing so you are already lowered the risk of trusting more "gray zone" CA's which might not take the rules for handing out certificates so strict (like for example, proper checking the identity of their clients). Secondly, because <b>*every*</b> certificate handed out by your trusted CA will be considered "authenticated", we'll close the loophole by issuing some extra checks.

Using WSS4J you can configure a matching pattern based on the subject DN property of the certificate. They have a nice blog entry on this here: <a href="http://coheigea.blogspot.ie/2012/08/subject-dn-certificate-constraint.html" target="_blank">http://coheigea.blogspot.ie/2012/08/subject-dn-certificate-constraint.html</a>.<br>We could specify that the DN of the certificate must match a given value like this:

<pre class="brush: java;">
Wss4jHandler handler = ... 
handler.setOption(WSHandlerConstants.SIG_SUBJECT_CERT_CONSTRAINTS, "CN = ...");
</pre> 

Note: that there is currently no setter for this using the Spring support for WSS4J in Wss4jSecurityInterceptor, so you'll have to extend it in order to enable this!

To conclude the steps being performed:  

<ol>
	<li>The certificate contained in the message is validated against the trusted CA in your trustore. When this validation succeeds it tells the application that the certificate is still valid and has actually been handed out by a CA that you consider trusted.<ul><li>This check gives us the guarantee that the certificate really belongs to the party that the certificate claimes to belong to. </li></ul>
	</li>
	<li>Optionally the certificate can also be checked on revocation so that we don't continue trusting certificates that are explictly been revoked.</li><li>WSS4J will check if some attributes of the certificate match the required values for the specific service (Subject DN Certificate Constraint support). <ul><li>This would be the authentication step; once the certficate has been found valid, we check if the owner of the certificate is the one we want to give access too</li></ul>
	</li>
	<li>Finally the signature in the message is verified by creating a new digest of the message, compare it with decrypted digest from the message and so forth</li>
</ol>

It should be noted that this check (at least when using WSS4J) is not done by default! If you don't specify it and simply add your CA's in the trust store you'll be leaving a security hole!

The second solution requires no extra configuration and depends on ONLY the certificate of the sender to be present in the truststore.<br>The certificate contained in the message is matched against the certificate in the truststore. If they match the sender is authenticated. There is no need to validate certificates against a CA since the certificates imported in the truststore are explicitly trusted (WSS4J will still check if the certificate is not expired and possibly check it for revocation). Again, there are no CA certificates (or CA intermediate certificates) in the truststore! Only the certificates of the senders that you want to give access too. Access is hereby controlled by adding (or removing) their certificate from the truststore.

This requires you to be cautious when initially importing the certificates since you'll have to make sure they actually represent the sender. But this is something you're always obliged to do when adding certificates to your truststore, also when adding CA certificates like in the first solution.

<b>Conclusion:</b> in the assumption you can limit the trusted CA's, the first solution is in most cases the preferred one and also the most scalable. For new clients there are no changes required to the truststore. The attributes to match can be stored externally so they are easy to change/add. Also, when the client certificates expires or gets revoked, you don't need to do anything special. The new certificate will we used by the sender at a given moment and will directly be validated against the CA in your truststore. In the second solution you would have to add the new certificate to the trustore and leave the old one in there for a while until the switch is performed.

Overall lessons learned: water tight security is hard. The #1 rule in IT (assumption is the mother of all f***ups) is certainly true here. Be skeptical and make sure you fully understand what is going on. Never trust default settings until you are sure what they do. The default setting on your house alarm (eg. 123456) is no good idea either. Neither is the default admin password on a Tomcat installation.
