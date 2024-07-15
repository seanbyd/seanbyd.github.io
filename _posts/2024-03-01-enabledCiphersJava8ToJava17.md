---
layout: post
title:  "Open Liberty enabledCiphers uses TLS_ in Java 17"
author: Sean Boyd
---

## Environment

| Category | Details  |
| :--- | :--- |
| Product  | WebSphere Liberty 24.0.0.1 |
| Java  | IBM Semeru Java 17 |
| Component | enabledCiphers |

## Problem summary

We recently deployed our first app to WebSphere Liberty using Java 17 - previously we only used Java 8. Immediately we started facing issues relating to SSL with the deployed application.

The development team didn't face similar issues but we, the admin team did. One point of note is that we apply security baselines to all WebSphere Liberty implementations, something the development teams do not. This allowed us to focus in on these differences.

## Diagnosis

To assist with these investigations we installed the adminCenter to see whether we could replicate the issue outside of the applicaiton itself. Luckily the adminCenter encountered the same problem.

This issue felt related to Java 17, and more specifically to our SSL settings (enabledCiphers).

The SSL config we historically used with Java 8 was similar to:

```xml
<ssl id="defaultSSLConfig"
    sslProtocol="TLSv1.2"
    keyStoreRef="defaultKeyStore"
    enabledCiphers="SSL_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384 SSL_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256 SSL_ECDHE_RSA_WITH_AES_256_GCM_SHA384 SSL_ECDHE_RSA_WITH_AES_128_GCM_SHA256"
/>
```

When the enabledCiphers keyword was removed, the adminCenter worked fine. When it was added back, the adminCennter stopped working. Rolling back from Java 17 to java 8 had the same effect - the adminCenter worked fine.

To help see what was happening internally, the following trace point was enabled.

```xml
<logging traceSpecification="SSLChannel=all:com.ibm.ws.ssl.*=all:com.ibm.websphere.ssl=all:com.ibm.wsspi.ssl.*=all" isoDateFormat="true" />
```

Upon replicating the issue the following error was found in the trace.log file. Removing each CipherSuite one-by-one resulted in a similar error being returned, until the keyword enabledCiphers was removed as it no longer contained any values.

````
java.lang.IllegalArgumentException: Unsupported CipherSuite: SSL_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
````

A brief search of the internet was performed to try to locate the enabledCiphers values but I have always found it challenging to find SSL CipherSuite lists. So, after a few minutes I gave up!

After removing the enabledCiphers keyword, I repeated the successful test and checked the trace.log file to locate the supported CipherSuite list. Voila! The CipherSuite keyword name had changed from an SSL_ prefix to a TLS_ prefix.

````
getEnabledCipherSuites Entry 
adjustSupportedCiphersToSecurityLevel Entry  
(49) TLS_AES_256_GCM_SHA384 TLS_AES_128_GCM_SHA256 TLS_CHACHA20_POLY1305_SHA256 TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384 TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256 TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256 TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256 TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 TLS_DHE_RSA_WITH_AES_256_GCM_SHA384 ...
````

## Solution

The solution was simply a matter of renaming the CipherSuite to use the TLS_ prefix. In the below example I took the chance to enabled TLS 1.3 for my testing and it worked fine.

```xml
<ssl id="defaultSSLConfig"
    sslProtocol="TLSv1.2,TLSv1.3"
    keyStoreRef="defaultKeyStore"
    enabledCiphers="TLS_AES_256_GCM_SHA384 TLS_AES_128_GCM_SHA256 TLS_CHACHA20_POLY1305_SHA256 TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384 TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256 TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256"
/>
```

**DISCLAIMER**

The enabledCiphers values were selected for testing purposes only. No recommendation for suitable and secure CipherSuites is inferred in the above example. You always need to consult your security experts and follow their guidance.

## Conclusion

I'm not sure whether any documentation exists around this change but in my researching I was unable to find any. Perhaps I didn't specify the right search keyword, but hopefully this blog shows not only the solution, but one way that can be followed to investigate similar problems.