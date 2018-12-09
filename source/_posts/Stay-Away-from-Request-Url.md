title: "Stay Away from Request.Url"
date: 2011-10-30 15:00:09
tags:
- ASP.NET MVC
- Misc
---

The title might be misleading but I will explain why we shouldn’t use the Request.Url in any asp.net application directly.

If you are writing an web application and you don’t know the environment of the server where it is getting deployed, then it is better not to use `Request.Url` it directly.

![](//static.rajeeshcv.com/images/2011/10/20111030073847_image_2.png)

This blog is running on a {% post_link New-Outlook–Rolling-out-My-Own-Blog-Engine "home grown blog engine" %}, which is written using asp.net MVC 3. For implementing some of the functionalities like generating sitemap.xml I had to get the root of the url(i.e. without any path). So I used the `Request.Url.GetLeftPart(UriPartial.Authority)` method and it worked perfectly on my local machine even when it is deployed my local machine IIS server. When I deployed my blog engine to [Appharbor] environment, the generated sitemap.xml has a URL with port number like *www.rajeeshcv.com:4566* instead of just *www.rajeeshcv.com*.

###Why?

After googling for some time I came across the solution posted [Appharbor] knowledge base - [workaround for generating absolute urls without port number][1]. In their environment there are load balancers which sits in front on the webserver, which uses a specific port number for contacting the server where the application is running. Below is a simple pictorial representation of that

![Network with load balancer](//static.rajeeshcv.com/images/2011/10/20111030073858_image_4.png)

So whenever we try to get `Request.Url`, application will get the actual request Url received by that webserver, which will have the port number also.

###How to solve it

Appharbor has provided a code snippet in the same article but it didn’t worked for me, I was still getting port number in the generated Url. My assumption is that, there is condition check for ignoring the local request so that it will work correctly in the local development environment.

```cs
if (httpContext.Request.IsLocal)
{
    uriBuilder.Port = httpContext.Request.Url.Port;
}
```

According to the MSDN documentation

> The `IsLocal` property returns true if the IP address of the request originator is 127.0.0.1 or if the IP address of the request is the same as the server's IP address.

In my case I think `IsLocal` was true (I really don’t the exact reason!!!). So instead of using the appharbor code snippet I came across a code from [FunnelWeb](http://www.funnelweblog.com/) which does the same (HttpRequestExtensions.cs)

Here is my version of that code

```cs
/// <summary>
/// Environments the specific request URL.
/// </summary>
/// <param name="request">The request.</param>
/// <returns></returns>
/// <remarks>
/// Code from FunnelWeb:  https://bitbucket.org/TheBlueSky/funnelweb/src/b64c74f361d3/src/FunnelWeb/Utilities/HttpRequestExtensions.cs
/// </remarks>
public static Uri EnvironmentSpecificRequestUrl(this HttpRequestBase request)
{

    UriBuilder hostUrl = new UriBuilder();
    string hostHeader = request.Headers["Host"];

    if (hostHeader.Contains(":"))
    {
        hostUrl.Host = hostHeader.Split(':')[0];
        hostUrl.Port = Convert.ToInt32(hostHeader.Split(':')[1]);
    }
    else
    {
        hostUrl.Host = hostHeader;
        hostUrl.Port = -1;
    }

    Uri url = request.Url;
    UriBuilder uriBuilder = new UriBuilder(url);

    if (String.Equals(hostUrl.Host, "localhost", StringComparison.OrdinalIgnoreCase) || hostUrl.Host == "127.0.0.1")
    {
        // Do nothing
        // When we're running the application from localhost (e.g. debugging from Visual Studio), we'll keep everything as it is.
        // We're not using request.IsLocal because it returns true as long as the request sender and receiver are in same machine.
        // What we want is to only ignore the requests to 'localhost' or the loopback IP '127.0.0.1'.
        return uriBuilder.Uri;
    }

    // When the application is run behind a load-balancer (or forward proxy), request.IsSecureConnection returns 'true' or 'false'
    // based on the request from the load-balancer to the web server (e.g. IIS) and not the actual request to the load-balancer.
    // The same is also applied to request.Url.Scheme (or uriBuilder.Scheme, as in our case).
    bool isSecureConnection = String.Equals(request.Headers["X-Forwarded-Proto"], "https", StringComparison.OrdinalIgnoreCase);

    if (isSecureConnection)
    {
        uriBuilder.Port = hostUrl.Port == -1 ? 443 : hostUrl.Port;
        uriBuilder.Scheme = "https";
    }
    else
    {
        uriBuilder.Port = hostUrl.Port == -1 ? 80 : hostUrl.Port;
        uriBuilder.Scheme = "http";
    }

    uriBuilder.Host = hostUrl.Host;

    return uriBuilder.Uri;
}
```

###Conclusion

IMHO this is a limitation with .Net framework itself because here we have to modify the behaviour of the framework class to achieve what we really want . If there is way in which the IIS web server can detect the topology on which it is running, then the `HttpRequest.Url` could be implemented correctly. So that the developer need not to worry about the deployment scenarios(at least in this simple case).

[Appharbor]: https://appharbor.com
[1]: https://support.appharbor.com/kb/getting-started/workaround-for-generating-absolute-urls-without-port-number
