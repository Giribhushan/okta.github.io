---
layout: docs_page
weight: 3
title: Choosing a Flow
excerpt: Choose an OAuth 2.0 or OpenID Connect flow
---

# Choosing a Flow

If you think of OAuth 2.0 as a delegated authentication framework, and OpenID Connect
as a high assurance, modern identity protocol layer over OAuth 2.0, you may still not be sure which flow to use.
Both OAuth 2.0 and OpenID Connect present a lot of options. We'll explain the options we've
implemented in Okta, and provide guidance around when to use which flow.

Authentication and authorization transactions are composed of many individual requests and responses.
Each set of transactions from end user to desired resource and back is called a flow. Each flow serves a specific scenario:

* Is the goal is to identify a user, or delegate authentication of a user to an app or other service?
* Is a user even involved in the transaction (user context)?
* What kinds of things is the authenticated user being given access to, and are they within or outside a trusted barrier?

With these factors in mind, use the following decision tree to choose a flow:

1. Are you working in a trusted environment? If no, go to next step. If Yes, choose password or client credentials flow. 
    a. Is there any human interaction? If yes, use password flow. If no, use client credentials flow.
    
2. Do you want to authenticate the user's credentials before authenticating them to access a resource? If No, go to next step. If Yes, choose one:
    a. Authorization code
    b. Implicit
    c. Hybrid

3. Do you simply want to refresh an existing token due to time limit or other barrier? Use Refresh flow. (not defined in specs?)

You also need to choose between OpenID Connect or Oauth 2.0. <Mysti needs help here>

Now that you know which flow to use, you can learn about the different elements Okta has implemented to make the flows work.

## A Few Basics

The [OAuth 2.0 spec](https://tools.ietf.org/html/rfc6749#section-1.3) and [OpenID Connect spec](https://openid.net/specs/openid-connect-core-1_0.html#Authentication) have defined a set of flows and the behaviors they should exhibit.
Okta models the key actors (users, apps or services, resources to be accessed) with properties, and then specifies the behavior of each property for each flow.

Essentially, these properties work together to create a token that is encoded with identity or access information that is passed on to the resource which an app or service
is trying to access on behalf of the user (or workflow in the case of service-to-service flows).

### Tokens and Related Properties

We said that properties serve as the model or stand-in for the various actors in an identity or delegated authentication flow.
At Okta, tokens are defined according to the OAuth 2.0 and OpenID Connect specs, and the actors involved are modeled by these properties:

* Applications or services are represented by the `application_type` property.
* The specific flow you choose is represented by the value of `grant_type`, or is defined implicitly. More about that later.
* Which token you receive (also part of a flow definition) is represented by the value of `response_type`.

#### Tokens

The problem with username/password credentials is that they are not scoped to time limits, resources, or any other limitation.
A token replaces the user (resource owner) credentials, and typically contains information about time limits, scoping (which resources
can be accessed with the token), and other information. These tokens are opaque, and thus thwart many man-in-the-middle attacks.

Think of the token as a hotel-room key: once you've authenticated yourself (like you do at the front desk), the token contains
the information you need to access the resources an app is requesting on your behalf.

#### App Types

In Okta, all apps and services are represented by a client application defined in Okta.

We support the following app types for OAuth 2.0 and OpenID Connect:

* Native (iOS, Android) an app on a smart phone or other mobile device
* Single-Page App (SPA)
* Web App: may be simple or complex
* Service App: Common for IoT applications or whenever one service needs to talk to another without a user context.

When you create an app in Okta, you'll choose one of these types, represented by [the `application_type` property](/docs/api/resources/oauth-clients.html#client-application-properties).

#### Grant Types and Flows

A grant type is the method a client app or service uses to obtain a token.
Okta supports the following `grant_type` values, most named after the flows defined in the OAuth 2.0 and OpenID Connect specs:

* Authorization code (`authorization_code`): the flow will use a code passed to it from the authorization endpoint to complete delegated authentication to the app or service.
* Implicit: I'm confused. Our docs indicate only OpenID Connect supports implicit, but the spec says it doesn't use it. Que?
* Hybrid: ???? Why don't we have a grant type for this? No Okta doc at all. What should I say about this?
* Password (`password`): Use only in a trusted environment. A login page collects a user's credentials, then passes them to the security token service.
* Client credentials (`client_credentials`): Use only in a trusted environment. For machine-to-machine access. This flow is OAuth 2.0 only, because OpenID Connect is an identity protocol. With no user context, no ID token is needed.
* Refresh token (`refresh_token`): You may need a refresh token for long-running flows. <Need more here> 

#### Response Types

There's one more variable to account for, the response type. The response type indicates what will be returned in the response:
an ID token, an access token, an authorization code, or some combination of the three.

#### Tokens Revisited

Okta manages four token types:

* ID token, for flows where the user needs to be identified. Don't send an ID token to an API.
* Access token, for flows where the user's access to a particular resource needs to be evaluated. Don't send an access token to identify a user.
* Refresh token, needed when the original token is in danger of expiring before the flow is complete.
* I think there are more....

You can request ID tokens with an OIDC flow, which requires no additional feature be enabled. You can request access tokens or ID tokens or both with an OAuth 2.0 flow, available when Okta's API Access Management feature is enabled.

