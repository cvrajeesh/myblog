title: "Integrating BrowserID In To Your ASP.NET MVC Application"
date: 2011-12-17 12:03:16
tags:
- ASP.NET
- Misc
---

*BrowserID* is a decentralized identity system which verifies the ownership of an email address in a secure manner, without the use of any application specific authentication mechanism. Which means, you don’t need to provide an login forms in your application, instead use BrowserID feature.

![BrowserId screenshot](/images/2011/12/20111217024113_BrowserID_4.png)

I am not going to explain in detail about this, but you can follow the links below to know more about it

* [Official website](https://browserid.org/)
* [How to Use BrowserID on Your Site](https://github.com/mozilla/browserid/wiki/How-to-Use-BrowserID-on-Your-Site)
* [How BrowserID Works](http://lloyd.io/how-browserid-works/)

I have created demo application to show how it could be integrated into ASP.NET MVC (it could applied to ASP.NET Forms also) application.

* [Demo app][demo]
* [Demo source code][code]


###How the Demo Application works

In this demo, [Secret page][1] link can only accessed if you have logged into the application. In order to login I have provided a *Sign in* button, like most of the applications, but when you click on it. It will open a pop-up window (make sure you have disable pop-up blockers), which is a URL from https://browserid.org not from my application. If you don’t have a BrowserID create one, otherwise enter the Email address and Password. Then follow the steps and finally click on the *Sign in* button, which log you into the application and from there you can access the [Secret page][1] link.

###How to implement this in ASP.NET MVC

**Enable BrowserID in your application : **

Include the BrowserID javascript file https://browserid.org/include.js to your master page
`<script src="@Url.Content("https://browserid.org/include.js")" type="text/javascript"></script>`


**Identify the User : **

Instead of displaying a login form on your site which takes a username and password, it uses the BrowserID JavaScript API when the user clicks your sign-in button. For that I have added the below script to the *_LogOnPartial.cshtml* which comes with the ASP.NET MVC application you have created

```js
$(document).ready(function () {
  $('#login').click(function () {
      navigator.id.getVerifiedEmail(gotVerifiedEmail);
  });
.....
.....
});
```

Upon a successful sign-in, you'll be called back with an assertion, a string containing a signed claim that proves the user is who they say they are. Which is passed to a method called `gotVerifiedEmail`

```js
// a handler that is passed an assertion after the user logs in via the
// browserid dialog
function gotVerifiedEmail(assertion) {
    // got an assertion, now send it up to the server for verification
    if (assertion !== null) {
        $.ajax({
            type: 'POST',
            url: '/Account/LogOn',
            data: { assertion: assertion },
            success: function (res, status, xhr) {
                if (res != null) {
                    loggedIn(res.email);
                }
            },
            error: function (res, status, xhr) {
                alert("login failure" + res);
            }
        });
    }
}
```

Which then sends a POST Request to the `LogOn` method on the Account controller, for verifying the assertion is correct or not. If it is a verified correctly, we will set up a forms authentication cookie so that ASP.NET feels that user has logged in to the application. Then returns the Email address back.

```cs
[HttpPost]
public ActionResult LogOn(string assertion)
{
    var authentication = new BrowserIDAuthentication();
    var verificationResult = authentication.Verify(assertion);
    if (verificationResult.IsVerified)
    {
        string email = verificationResult.Email;
        FormsAuthentication.SetAuthCookie(email, false);
        return Json(new { email });
    }

    return Json(null);
}
```

In order to do the verification, we post the assertion to the URL provided by the Identity Authority itself (https://browserid.org/verify in this case), which will give a valid response if it is valid. The Verify method looks like this

```cs
public VerificationResult Verify(string assertion)
{
    JObject verificationResult = this.GetVerificationResult(assertion);
    string email = "";
    bool verified = false;

    if (verificationResult != null && verificationResult["status"].ToString() == "okay")
    {
        email = verificationResult["email"].ToString();
        verified = true;
    }

    return new VerificationResult {Email = email, IsVerified = verified };
}

/// <summary>
/// Gets the verification result from Identity Authority.
/// </summary>
/// <param name="assertion">The assertion.</param>
/// <returns></returns>
private JObject GetVerificationResult(string assertion)
{
    // Build he request
    var req = (HttpWebRequest)WebRequest.Create(IdentityAuthorityVerificationUrl);
    req.Method = "POST";
    req.ContentType = "application/x-www-form-urlencoded";

    if (HttpContext.Current == null)
    {
        throw new NullReferenceException("HttContext is null. Please make sure that, your application is running in a web scenario");
    }

    // Build the data for posting
    var payload = string.Format("assertion={0}&audience={1}", assertion, HttpContext.Current.Request.Headers["Host"]);
    byte[] payloadInBytes = Encoding.UTF8.GetBytes(payload);
    req.ContentLength = payloadInBytes.Length;

    var dataStream = req.GetRequestStream();
    dataStream.Write(payloadInBytes, 0, payloadInBytes.Length);
    dataStream.Close();

    JObject result = null;

    var res = req.GetResponse();
    dataStream = res.GetResponseStream();
    if (dataStream != null)
    {
        var responseData = new StreamReader(dataStream).ReadToEnd();
        res.Close();
        dataStream.Close();
        result = JObject.Parse(responseData);
    }else
    {
        res.Close();
    }

    return result;
}
```

Hope this will help you to setup an authentication system to your application very easily and in a more secure way.

[Demo]: http://nbrowserid.apphb.com/
[Code]: https://github.com/cvrajeesh/NBrowserID
[1]: http://nbrowserid.apphb.com/Secret
