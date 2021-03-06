# Introduction

> Access to fetch at 'https://www.yourapi.com' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.

If you have ever built a web app and wants to interact with a REST API, you might be familiar with this error. If you also build the API, you might wonder: why it worked with Postman when you're testing your API, it may even work with mobile apps as well. _There must be something wrong with my web app._ However, it is not the case.

This article will give you a overall idea on what CORS is, and how to '_fix_' it.

# CORS

## What is CORS

**CORS** stands for Cross-Origin Resource Sharing. It is a mechanism that restricts requests coming from a different domain.

## Why CORS

The rationale behind it is to allow the server (API) on one origin to restrict behavior for other origins, since one may only want to allow others to read the data, but not to modify the data at will. As a result, requests like **GET** are usually allowed by default. However, **PUT**, **DELETE** and sometimes **POST** would be restricted.

## How does CORS work

When the browser is about to send a request to a different origin, one that will trigger the CORS, e.g. a **PUT** request, it will not send the request itself immediately. Instead, it will send what's called a _preflight_ request, which serves as a _test_ whether the server allows the communication. If the server permits, in response to the preflight request, the browser will then send the actual request. Otherwise, it will reject with 405 and the error message shown at the start.

## References

We provided a very brief explanation for CORS, for those who wants to dig deeper, here's a thorough explanation on CORS: [https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS).

Another good read would be [What is CORS?](https://www.codecademy.com/articles/what-is-cors)

# How to make it work

The CORS issue can be very annoying, especially for learners. This article will sum up the solutions which may save your hours of search on _StackOverflow_. The solution depends on your scenarios, more specifically, how much control you have over the server. We will start from the _best_ scenario to the _worst_.

## Scenario 1: full control over the server

If you are the one that deploys the API onto your own server, then you are in the best scenario. All you need to do is to configure your server, assuming that you are using **Nginx**. If not, you probably should. But nonetheless, you can still check out the next scenario which may also fix this issue.

Here's a code snippet from [https://enable-cors.org/server_nginx.html](https://enable-cors.org/server_nginx.html) which shows how to configure a wide-open CORS policy.

```
if ($request_method = 'OPTIONS') {
  add_header 'Access-Control-Allow-Origin' '*';
  add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';

  # Custom headers and headers various browsers *should* be OK with but aren't
  add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';

  # Tell client that this pre-flight info is valid for 20 days
  add_header 'Access-Control-Max-Age' 1728000;
  add_header 'Content-Type' 'text/plain; charset=utf-8';
  add_header 'Content-Length' 0;
  return 204;
}
if ($request_method = 'POST') {
  add_header 'Access-Control-Allow-Origin' '*';
  add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
  add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';
  add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
}
```

You may tailor it to your need. For example, instead of using a wildcard for all origins, you can only permit your web app domain. And you can also only allow more specific methods and headers to go through.

This solution allows you to configure CORS without tempering your API code, which can be ideal for most cases. However, if this is not want you want, or, more often, you do not have control over the server, say you deployed your API via **Heroku**, which is quite convenient and worry-free if you don't want to care about the server setup miscellaneousness, you may find your solution in the next scenario.

# Scenario 2: my API but not my server

In this scenario, you fully deployed your API, but you might be using some hosting service like **Heroku** or alike thus do not have control over the server configurations. This section guides you through adding that needed header through your backend code.

As there is no universal preference on backend language and framework, we can only pick one of our favorite, but it should be a popular choice and should at least shed light on how to solve it in your case.

We used a **Flask** app for demonstration, which is a very popular **Python** framework, and sorry **Django** lovers ;)

```
app = Flask(__name__)

@app.after_request
def after_request(response):
    response.headers.add('Access-Control-Allow-Origin', '*')
    response.headers.add('Access-Control-Allow-Headers', 'Content-Type')
    return response
```

The above code block shows how to add the additional header after the request is being handled and before the response is sent. Same ideas here, you can configure it to not be so wide-open and only allow certain origins, methods or headers.

# Scenario 3: I'm just a web developer

No offense to front-end developers, but you might be in the 'worst' scenario here. However, it is not your fault and it is quite common. You might just want to interact with a third party API and it just keeps complaining about CORS. We will talk about the possible solutions here.

The ideal solution would be inform the API owner about this issue, since technically this is **their fault**. However, you might not get an immediate response. What's worse, you might encounter an arrogant backend developer educating you what a REST API is and that it should act identically to any clients. Since they found no issue with using it, it's your own misconfiguration. Please refer them to this blog if this is your situation. However, in the meantime, you may try the next solution.

Another solution would be building a proxy API that servers as a bridge to the third party API. It sounds a bit hacky but it might be the only working solution you can have if you wish your web app to ship and work. The proxy API receives the requests from your web app and send it to the third party API, then respond with what's received there. Since it doesn't go through the browser, it circumvent the CORS issue You can then configure your own API as in the previous two scenarios.

The last solution would be using browser plugins to inject the needed headers into the response. This is not recommended since it's only 'fooling yourself', and it would not work on other computers, for instance, your users'. However, it is still useful if you're testing things around locally and would like to see some results ASAP. There are usually plenty of such plugins for each browser. One of them is the [Allow-Control-Allow-Origin](https://chrome.google.com/webstore/detail/allow-control-allow-origi/nlfbmbojpeacfghkpbjhddihlkkiljbi) extension for **Chrome**.

There are also tutorials on how to turn of the CORS check on browsers, but we would not cover it here since it's not scalable either.

# Conclusion

This article provided a very high-level view on the CORS policy and there're a lot more to learn, which can be found in the reference section. We tailored the solutions to different scenarios to help you find the solution that suits your need. We hope you find this article helpful, since I've personally encounter these scenarios and wished that there had been such documentations describing the reasons and solutions. Cheers!
