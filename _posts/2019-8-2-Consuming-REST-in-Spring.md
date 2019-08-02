---
layout: post
title: Consuming REST in Java using the Spring Framework
---

I have done backend Java development work, and entirely all of it has been in [Spring](https://spring.io/), but it continues to amaze me the amount of features Spring has right out of the box. This week I learned about how to use Spring to consume a REST endpoint.

Anyone who has used Java but hasn't messed around with Spring knows how finicky the whole process is. Generally, you need to use a [HttpURLConnection](https://docs.oracle.com/javase/7/docs/api/java/net/HttpURLConnection.html) or equivalent. And then from that, you will need to perform a bunch of GET/POST requests. It's tedious and error-prone to set up for all your API endpoints. Fortunately for us, Spring offers something out of the box that makes this process incredibly trivial, while allowing us to modify an object by the end of it, and this resulted in 11 lines of code in the worst-case scenario.

I needed to get a list of objects from a JSON endpoint, but at the same time pass in an auth key with a custom header. The [Spring example](https://spring.io/guides/gs/consuming-rest/) on how to get this done was nearly there but it didn't meet my needs. After a bit of sleuthing, I did the following:

~~~java
RestTemplate template = new RestTemplate();
HttpHeaders headers = new HttpHeaders();
headers.set("secret-api-key", apiKey);
HttpEntity<List<Customer>> entity = new HttpEntity<>(headers);
ResponseEntity<Customer[]> response = template.exchange(url, HttpMethod.GET, entity, Customer[].class);
 
if (response.getStatusCode().is2xxSuccessful() && response.hasBody()) {
 // do something with response.getBody()
} else {
 // failure path
}
~~~

In the end, since I figured it would be better to reuse the core code but allow GET/POSTs and custom types, I modified the above code into the following:

~~~java
private <T> ResponseEntity<T> baseCall(String endUrl, HttpMethod method, Class<T> clazz) {
 RestTemplate template = new RestTemplate();
 HttpHeaders headers = new HttpHeaders();
 headers.set("secret-api-key", apiKey);
 HttpEntity<Void> entity = new HttpEntity<>(headers);
 return template.exchange(baseUrl + endUrl, method, entity, clazz);
}
~~~

Where `baseUrl` was the starting url (e.g. `https://softwareendpoint.com/api/v1`), `endUrl` was the end path (e.g. `/customers/larry`) and `clazz` being the class type returned.

The only thing I had to be careful about was the data was returned as an array and not a list, but I used Guava's [`ImmutableList.copyOf()`](https://guava.dev/releases/21.0/api/docs/com/google/common/collect/ImmutableList.html#copyOf-E:A-) method to achieve this effect.

I suspect I might have to modify the above code in the future so that I handle the POST case of sending JSON body back, in which case the `HttpEntity<Void>` object type will change to the POST object type.




