title: "One of the Top E-Commerce Website in India Stores Passwords Incorrectly"
date: 2012-12-06 12:36:44
tags:
---

This incident made me feel very bad – couple of weeks before I ordered a product from [naaptol] a well-known E-Commerce website in India.  It was the first time I was ordering something from them. So I had to register first, after the registration was completed got a confirm email from them

![Registration confirmation email from naaptol](http://rajeesh.cdn.rhyble.com/images/2012/12/20121206105945_naaptolregistration.png)

Oops!!! My password is in clear text.

I thought they might be generating the email during the registration process before it is stored in their database. Without bothering much, I ordered the item. Thought about investigating about the “clear text password” little bit, but somehow forgot it.

After few days when I got the product which I have ordered, it reminded me about this password incident. So in order see whether they are hashing my password or not, I used their **Forgot my password** option. It asked for my email address. After the submission got a message saying

> We' have sent you an e-mail at the submitted ID including instructions. You'll be back to your shopping place in no matter of time.

As expected, got an email from them

![Forgot my password email from naaptol](http://rajeesh.cdn.rhyble.com/images/2012/12/20121206105945_naaptolforgotpassword.png)

It reads *Here is your new Login and Password* and surprisingly password they gave is same as my old password even though in the email it is mentioned that they are sending a new password. It confirmed that they are not hashing the passwords.

Will they be storing it in plain text? Who knows…

> Did someone tell you internet is not a good place to store your secrets?

I tried to play a nice role, so sent them an email telling about about the password hashing problem. You know what happened… no reply from them till now.

If you are in the naaptol technical team, convince your boss about the importance of securing the password and push that functionality in the next release.

If you are a non-technical manager in naaptol, tell your developers to read this article from Jeff - [You're Probably Storing Passwords Incorrectly](http://www.codinghorror.com/blog/2007/09/youre-probably-storing-passwords-incorrectly.html)

[naaptol]: http://www.naaptol.com/
