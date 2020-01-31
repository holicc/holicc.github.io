## Background 

Recently,out team have to divided original system to two microservice system. let's call they system A and system B.My goal is to implement a **Authorization Server** and Single-Sign-On. So i decided to use Spring security framework to do this.It's not easy as i think,i got lots of problems.Finally i made it,and i want to recored this to review.

## What is [OAuth 2.0](https://oauth.net/2/) 

OAuth is not an API or a service: it’s an open standard for authorization and anyone can implement it.OAuth 2.0 is widely used by applications.For example,a app need access your wechat contacts list,so the app need authorized by wechat,you have probably used OAuth 2.0.

OAuth 2.0 includes four roles:

 -  **Resource owner** (user's private resources such as profile,photos etc)
 -  **Client** (the app want to access user's resources)
 -  **Resource server** (stores user’s private resources and shares them with authorized clients)
 -  **Authorization Server** (used for certification)

OAuth 2.0 support four grant types to obtain the resource owner's permission(called **access_token**)
 - **Password Credentials** 
 - **Client Credentials**
 - **Authorization Code**
 - **Implicit**

*Password Credentials* usually used for trusted clients
> used by first-party clients to exchange a user's credentials for an access token.

*Client Credentials* used to access resources owned by the client itself
> typically used by clients to access resources about themselves rather than to access a user's resources.

**Above two grant types will revealed resource owner's credentials to third-part**

*Authorization Code* the most complicate flow in OAuth 2.0
> used by confidential and public clients to exchange an authorization code for an access token.After the user returns to the client via the redirect URL, the application will get the authorization code from the URL and use it to request an access token.

*Implicit* generally not recommended to use the implicit flow 
>a simplified flow that can be used by public clients, where the access token is returned immediately without an extra authorization code exchange step.

**Above two grant type are more interesting as they are used by public clients and users give their permission to third party applications**

In my practice,i only used *Authorization Code* and *Password* grant types , finally choose the *Password* grant type because own system A and system B is trusted clients. 

## Why using OAuth 2.0

Actually,i'm not sure why i have to using the OAuth 2.0.I just Google the Spring security want to make a Single-Sign-On system, and then the google search resutls show me lots of anwsers like : *Spring Security SSO With OAuth 2.0* ~~as far now i just know OAuth 2.0 is a protocol~~OAuth 2.0 is not an authentication protocol i have to use it if i using the Spring Security framework.So i’m thinking **Why using OAuth 2.0?** and **Why OAuth 2.0 is safe?**

i found a simple anwser of the *What goal of the OAuth 2.0?* on offical site.

> OAuth 2.0 focuses on client developer simplicity while providing specific authorization flows for web applications, desktop applications, mobile phones, and living room devices.

**Simplicity** is one of the keypoint for *why using OAuth 2.0*,becasue OAuth 1.0 implementation attempts are caused by the complexity of the cryptographic requirements of the specification.Anothrer reason is OAuth 2.0 support support for non-browser based applications.

**OAuth 2.0 Access tokens are "short-lived"** which means OAuth 2.0 is more safe than OAuth 2.0 , when access_token is expired you need refresh_token to get a new access_token.*You should notice that access token can't be revoked you just have to wait for them to time out.*

**Not interact with user credentials** no need user's username or password for access_token instead using authoirzation code exchange for access_token.

**Keep user's infomation in token** It accesses the data using tokens instead of using their credentials and stores data in online file system of the user such as Google Docs or Dropbox account. 

## Conclusion

Actually,OAuth 2.0 purpose is to *“How can I allow an app to access my data without necessarily giving it my password?”* , and support non-browser based applications.OAuth 2.0 security depends on how you implement system is.


## Reference

 - [Official Site](https://oauth.net)
 - [Introducing OAuth 2.0](https://hueniverse.com/introducing-oauth-2-0-b5681da60ce2)
 - [What the Heck is OAuth?](https://developer.okta.com/blog/2017/06/21/what-the-heck-is-oauth)
 - [How is OAuth 2 different from OAuth 1?](https://stackoverflow.com/questions/4113934/how-is-oauth-2-different-from-oauth-1)
