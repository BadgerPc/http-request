# http-request

This lib built on apache http client for sending rest requests.

**http-request** Features: 
To build your HttpRequest requires no more than 5 minutes. <br/>
Used builder pattern to create HttpRequest. <br/>
HttpRequest design as one instance to one URI. <br/>
HttpRequest is an immutable (thread safe after build). <br/>
Overloaded methods execute() for sending request. <br/>
Wrapped all exceptions, If connection failure -> status code is a 503(SC_SERVICE_UNAVAILABLE),
                                                If failed deserialization  of response body -> status code is a 502(SC_BAD_GATEWAY). <br/>
After request provide ResponseHandler instance by many methods to manipulate response data. <br/>
Supported converting response to the type which you want. <br/>
Supported ignore response body if you interested only status code. <br/>
Supported converting from Json. <br/>
Supported converting from Xml. <br/>
Optimized performance. <br/>

Full API documentation is available [here](http://javadoc.io/doc/com.jsunsoft.http/http-request).


**Note: HttpRequest objects are immutable they can be shared after build.**

### How to use

For build HttpRequest by default options

```java
HttpRequest<SomeTypeToConvertResponseBody> httpRequest = RestClient.createGet(uriString,  SomeTypeToConvertResponseBody.class).build();
```
If you want ignore the convert of response body, you must build so:
```java
HttpRequest<?> httpRequest = RestClient.createGet(uri).build();
```
If you want convert response body to Generic class (example List<T>) by some type you must build so:

```java
HttpRequest<List<SomeType>> httpRequest = RestClient.createGet(uri,  new TypeReference<List<SomeType>>(){}).build();
ResponseHandler<List<SomeType>> responseHandler = httpRequest.execute();

List<SomeType> someTypes = responseHandler.get(); //see javadoc of get method
or
List<SomeType> someTypes = responseHandler.orElse(Collections.emptyList());
```

Send simple http request.
```java
Perform a GET request and get the status of the response
HttpRequest<?> httpRequest = HttpRequestBuilder.createGet("https://www.jsunsoft.com/");
int responseCode = httpRequest.execute().getStatusCode()
```

```java
Build HttpRequest and  add HEADERS which should be send always. Example:
HttpRequestBuilder.create(HttpMethod.PUT, "https://www.jsunsoft.com/").addDefaultHeader(someHeader).build();
HttpRequestBuilder.create(HttpMethod.PUT, "https://www.jsunsoft.com/").addDefaultHeader(someHeaderCollection).build();
HttpRequestBuilder.create(HttpMethod.PUT, "https://www.jsunsoft.com/").addDefaultHeader(someHeaderArray).build();
HttpRequestBuilder.create(HttpMethod.PUT, "https://www.jsunsoft.com/").addDefaultHeader(headerName, headerValue).build();
```

By default connection pool size of apache http client is 2. I changed the parameter to default value 128. For set custom value you can:
```java
HttpRequestBuilder.create(someHttpMethod, someUri).maxPoolPerRoute(someIntValue).build();
or
ConnectionConfig connectionConfigInstance = ConnectionConfig.create().maxPoolPerRoute(someIntValue);
HttpRequestBuilder.create(someHttpMethod, someUri).connectionConfig(connectionConfigInstance).build();
```

How to set proxy. Example:

```java
Proxy proxy = new Proxy(host, port)
HttpRequest httpRequest = HttpRequestBuilder.create(someHttpMethod, someUri).proxy(proxy).build()
```
Default timeouts.
```text
socketTimeOut is 30000ms
connectionRequestTimeout is 30000ms
```
For more information see (https://hc.apache.org/httpcomponents-client-ga/tutorial/html/connmgmt.html)
or (http://www.baeldung.com/httpclient-timeout)

To change default timeouts you can:
```java
HttpRequestBuilder.create(someHttpMethod, someUri)
                                .connectTimeout(intValue)
                                .socketTimeOut(intValue)
                                .connectionRequestTimeout(intValue).build();
```



**Real world example how http-request simple and useful**.

No try/catch, No if/else

```java
import com.jsunsoft.http.*;
import java.util.List;
import java.nio.charset.Charset;

public static class Rest{
    private static final HttpRequest<List<String>> httpRequest =
     HttpRequestBuilder.createGet("https://www.jsunsoft.com/", new TypeReference<java.util.List<String>>() {})
     .contentTypeOfBody(ContentType.create("application/json", Charset.forName("UTF-8"))).build(); //it is used by default 
     
     public void send(String jsonData){
         httpRequest.executeWithBody(jsonData).ifSuccess(this::whenSuccess).otherwise(this::whenNotSuccess);
     }
     
     private void whenSuccess(ResponseHandler<List<String>> responseHandler){
         //When predicate of filter returns true, calls whenHasResult else calls whenHasNotResult
         responseHandler.filter(ResponseHandler::hasResult).ifPassed(this::whenHasResult).otherwise(this::whenHasNotResult);
     }
     
     private void whenNotSuccess(ResponseHandler<List<String>> responseHandler){
         //For demo. You can handle what you want
          System.err.println("Error code: " + responseHandler.getStatusCode() + ", error message: " + responseHandler.getErrorText());
     }
     
     private void whenHasResult(ResponseHandler<List<String>> responseHandler){
         //For demo. 
         List<String> responseBody = responseHandler.get();
         System.out.println(responseBody);
     }
     
     private void whenHasNotResult(ResponseHandler<List<String>> responseHandler){
         //For demo. 
           System.out.println("Response is success but body is missing. Response code: " + responseHandler.getStatusCode());
     }
}
```

To use from maven add this snippet to the pom.xml `dependencies` section:

```xml
<dependency>
  <groupId>com.jsunsoft.http</groupId>
  <artifactId>http-request</artifactId>
  <version>0.0.2</version>
</dependency>
```

Pull requests are welcome.