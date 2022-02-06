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
OAuth 2.0 specification defines five main grant types - authorization code, implicit, 
resource owner password credentials, client credentials, and refresh token.
It also provides an extensibility mechanism for defining additional types.

### Scopes

### Tokens
