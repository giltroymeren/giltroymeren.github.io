---
layout: post
title:  "Setting timeout to 'ClientHttpRequestFactory' with custom 'HttpClient' causes 'UnsupportedOperationException'"
date:   2016-11-11
categories: java notes
---

I have a project in my company that needs a connection timeout set to an instance of `ClientHttpRequestFactory` with a custom `HttpClient` because I need to bypass keys. This is used for Spring MVC's [`RestTemplate`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html) connection.

## Problem

Here is the code for creating the custom `ClientHttpRequestFactory`:

~~~ java
public ClientHttpRequestFactory getClientHttpRequestFactory()
    throws KeyManagementException, NoSuchAlgorithmException, KeyStoreException
{
    HttpComponentsClientHttpRequestFactory requestFactory =
        new HttpComponentsClientHttpRequestFactory();
    requestFactory.setHttpClient(httpClientFactory.get());
    requestFactory.setConnectTimeout(1000);
    return requestFactory;
}
~~~

I get the following error messages when `setConnectTimeout()` is present **after** `setHttpClient()`:

    Exception in thread "pool-1-thread-3" Exception in thread "pool-1-thread-1" java.lang.UnsupportedOperationException
    Exception in thread "pool-1-thread-2"  at org.apache.http.impl.client.InternalHttpClient.getParams(InternalHttpClient.java:204)
     at org.springframework.http.client.HttpComponentsClientHttpRequestFactory.setConnectTimeout(HttpComponentsClientHttpRequestFactory.java:118)

I experimented with other ways to solve it and found these two OK cases:

1. without `setConnectTimeout()`

    I can connect and bypass with my custom `HttpClient` successfully however time is sacrificed.

2. without `setHttpClient()`

    The connection is faster but I get bypass errors which is useless since I cannot connect to anything.

        PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target

Regarding `HttpComponentsClientHttpRequestFactory.java:118`, I looked at it was a deprecated `getHttpClient().getParams()` issue with `HttpClient` version `4.3.x` but when I tried to upgrade to the latest `4.5.2` (as of 2016 February) I still get `UnsupportedOperationException`.

## Solution

Since `HttpClient`'s `getParams()` is deprecated:

    Exception in thread "pool-1-thread-2"  at org.apache.http.impl.client.InternalHttpClient.getParams(InternalHttpClient.java:204)

We should use the suggested `RequestConfig` for versions 4.3.2 and up (based from Zamanillo's answer).

In my case, instead of setting the connection timeout in an instance of `HttpComponentsClientHttpRequestFactory`, I did it in the custom `HttpClient`:

~~~ java
public HttpClient get()
    throws KeyManagementException, NoSuchAlgorithmException, KeyStoreException
{
    HttpClientBuilder httpClientBuilder = HttpClients.custom();

    CloseableHttpClient httpClient =
        httpClientBuilder.setDefaultRequestConfig( getBuilder().build() )
            .setSSLSocketFactory( <custom bypass> ).build();
    return httpClient;
}
~~~

~~~ java
private Builder getBuilder()
{
    Builder builder = RequestConfig.custom();
    builder.setConnectTimeout( 500 );
    return builder;
}
~~~

I can use both the timeout and bypass in this case.

## References

* Zamanillo, Víctor. "HttpClient.getParams() deprecated. What should I use instead?" *Java - HttpClient.getParams() deprecated. What should I use instead? - Stack Overflow*. N.p., 26 Feb. 2014. Web. 17 Nov. 2016. \<[`http://stackoverflow.com/questions/22038957/httpclient-getparams-deprecated-what-should-i-use-instead/22041217#22041217`](http://stackoverflow.com/questions/22038957/httpclient-getparams-deprecated-what-should-i-use-instead/22041217#22041217)\>.