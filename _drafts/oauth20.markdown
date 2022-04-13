---
layout: post
title: "OAuth 2.0 and OpenID Connect"
---

https://aaronparecki.com/2019/12/12/21/its-time-for-oauth-2-dot-1

# OAuth Actors

OAuth defines four roles:

### Resource Server

The resource server hosts protected data.
Web service is a resource server. 
It decides who can receive data based on access tokens.

### Client

The client is an application that consumes protected resources on the end user's behalf.
It could be any kind of application: web, mobile, desktop or other. 
Single Page Application is an example of the client.

### Resource Owner

The resource owner is the end user who owns the resources.
Usually it's a person who allows the client application to access his resources on his behalf.<br>
The client application can consume resources with no end user. 
In such cases the client application itself is the end user.

### Authorization Server

The authorization server issues access token to the client application after successfully
authenticating the end user and checking identity of the client application. 
The client application uses this token to access protected resources on behalf of the end user.
The resource server asks the authorization server to validate token received with a client request.

# OAuth components

### Grant types

Process when a client receives a token from an authorization server is known as **grant** in OAuth 2.0.

OAuth 2.0 specification defines five main grant types

- Authorization code
- Implicit
- Resource owner password credentials
- Client credentials
- Refresh token

It also provides an extensibility mechanism for defining additional types.

### Scopes

### Tokens



In OAuth 2.0, this dichotomy is addressed by removing the requirement for all clients to have a client secret 
and instead defining two classes of clients, public clients and
confidential clients, based on their ability to keep a configuration time secret.

why not use implicit flow https://developer.okta.com/blog/2019/08/22/okta-authjs-pkce

attack that PKCE prevents
- https://datatracker.ietf.org/doc/html/rfc7636
- https://docs.wso2.com/display/IS520/Mitigating+Authorization+Code+Interception+Attacks

Browser-Based Apps
- https://oauth.net/2/browser-based-apps/
- Spec https://datatracker.ietf.org/doc/html/draft-ietf-oauth-browser-based-apps

js client
- https://www.npmjs.com/package/@openid/appauth
- https://www.npmjs.com/package/angular-auth-oidc-client
- https://www.npmjs.com/package/oidc-client

Oidc two tokens reason http://www.thread-safe.com/2011/11/openid-connect-tale-of-two-tokens.html?m=1

auth0 spa lessons https://auth0.com/docs/identity-labs/lab-4-single-page-app/identity-lab-4-exercise-1

oidc official libraries https://openid.net/developers/libraries/

IDP stands for Identity Provider, a party that offers user authentication as a service. 
RP stands for Relying Party, an app that outsources its user authentication function to an IDP. 
Relying because Client app relies on IDP in authentication questions.

oidc spec adds params to Authentication Request
- nonce - String value used to associate a Client session with an ID Token? Что за сессия?
- display(page/popup),
- prompt - string values that specifies whether the Authorization Server prompts the End-User for reauthentication and consent. 
'none' - An error is returned if an End-User is not already authenticated. 
Как сервер узнает какой именно пользователь делает запрос и что уже аутентифицирован??
- id_token_hint, login_hint

oidc Offline Access - Refresh Token can be used to obtain an Access Token that grants access to the End-User's UserInfo 
Endpoint even when the End-User is not present (not logged in). По умолчанию этот режим отключен, 
т.е. при обновлении токена сервер всегда проверяет залогинен ли юзер? Если да - как сервер это проверяет?



вопрос на стэковерфлоу - зачем id токену нужен срок действия и как он соотносится со сроком действия  access токена, 
что будет если  id токен протух раньше access токена? 
https://stackoverflow.com/questions/25686484/what-is-intent-of-id-token-expiry-time-in-openid-connect

сессии
- oidc what is silent Authentication/silent token renew??
- prompt=none как работает?
- oidc сессии - что это такое, как работают?
- Authentication Request nonce параметр связан с сессий??
