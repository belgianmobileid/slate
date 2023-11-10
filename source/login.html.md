---
title: itsme® OpenID Connect documentation

language_tabs: # must be one of https://git.io/vQNgJ
 - http: 

toc_footers:
 - <a href='#'>Sign Up for a Developer Key</a> -->
 - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

search: true
---
# 1. Introduction
<b>This documentation is deprecated. Please refer to it ONLY if you are still using our version 1 OIDC interface. Any new partner will be using our version 2, which is documented on <a href="https://belgianmobileid.github.io/doc/authentication/">https://belgianmobileid.github.io/doc/authentication/</a>.</b>

itsme® is a trusted identity provider allowing partners to use verified identities for authentication and authorization on web desktop, mobile web and mobile applications. 

The objective of this document is to provide all the information needed to integrate the **Login** service using the <a href="http://openid.net/specs/openid-connect-core-1_0.html" target="blank">OpenID Connect Core 1.0 specifications</a>.

# 2. Integration guide

Our itsme® app can be seamlessly be integrated with your web desktop, mobile web or mobile application so you can perform secure identity checks.

**Technical overview**

itsme® integration is based on the <a href="http://openid.net/specs/openid-connect-core-1_0.html#CodeFlowAuth" target="blank">Authorization Code Flow</a> of OpenID Connect 1.0. The Authorization Code Flow goes through the steps as defined in <a href="http://openid.net/specs/openid-connect-core-1_0.html#CodeFlowSteps" target="blank">OpenID Connect Core Authorization Code Flow Steps</a>, depicted in the following diagram:
  
 ![Sequence diagram describing the OpenID flow](OpenID_Login_SeqDiag.png)

<aside class="notice">OpenID Connect provides certified libraries, products, and tools which could help you integrating the itsme® service. For more information, please visit the official webpage: <a href="https://openid.net/developers/libraries/" target="blank">https://openid.net/developers/libraries/</a>.
</aside>
 
**How it works**

<ol>
  <li>The User indicates on your end he wishes to authenticate with itsme®</li>
  <li>Your web desktop, mobile web or mobile application (aka 'Relying Party' in the OpenID Connect specification) sends a request to itsme® (aka 'OpenID Provider' in the OpenID Connect specification) to authenticate the User. This request will redirect the User to the itsme® Front-End. itsme® then authenticates the User by asking him
    <ul type>
      <li>to enter his phone number on the itsme® OpenID web page</li>
      <li>authorize the release of some information’s to your application</li>
      <li>to provide his credentials (itsme® code or fingerprint or FaceID)</li>
    </ul>
  
  If you are building a mobile web or mobile application, the User don’t need to enter his mobile phone number on the itsme® OpenID web page, he will be automatically redirected to the itsme app via the Universal links or App links mechanism.</li>
  <li>Once the User has authorized the request and has been authenticated, itsme® will return an Authorization Code to the Service Provider Front-End, redirecting the user to your mobile or web application.</li>
  <li>The Service Provider Back-End calls the itsme® Token Endpoint and exchanges the Authorization Code for an ID Token identifying the User and an Access Token.</li>
  <li>The Service Provider Back-End MAY request the additional User information from the itsme® userInfo Endpoint by presenting the Access Token obtained in the previous step.</li>
  <li>At this stage you are able to confirm the success of the operation and display a success message.</li>
</ol>

If a user doesn't have the itsme® app, they'll be redirected to a mobile website with more information and download links.


<a name="OpenIDConfig"></a>
## 2.1. Check itsme® OpenID Provider configuration

To simplify implementations and increase flexibility, <a href="https://openid.net/specs/openid-connect-discovery-1_0.html" target="blank">OpenID Connect allows the use of a Discovery Document</a>, a JSON document containing key-value pairs which provide details about itsme® configuration, such as the URIs of the 

<ul>
  <li>Authorization, Token and userInfo Endpoints</li>
  <li>supported claims</li>
  <li>JWKSet URL</li>
  <li>...</li>
</ul>

The Discovery document for itsme® can be retrieved from: 

Environment | URL
:-------- | :--------
**SANDBOX** | <a href="https://e2emerchant.itsme.be/oidc/.well-known/openid-configuration" target="blank">https://e2emerchant.itsme.be/oidc/.well-known/openid-configuration</a>
**PRODUCTION** | <a href="https://merchant.itsme.be/oidc/.well-known/openid-configuration" target="blank">https://merchant.itsme.be/oidc/.well-known/openid-configuration</a>


## 2.2. Create a itsme® button on your application

First, you will need to create a button to allow your users to authenticate with itsme®. See the <a href="https://brand.belgianmobileid.be/d/CX5YsAKEmVI7/documentation#/ux/buttons-1518207548" target="blank">Button design guide</a> before you start the integration. 

Upon clicking this button, we will open a modal view which contains a field that need to be filled by the end user with it’s phone number. Note that mobile web users will skip the phone number step, as they use the itsme® mobile app directly to authenticate.


## 2.3. Crafting your client authentication method

Some itsme® endpoints require client authentication in order to protect entitlement information between interested parties. 

The OpenID Connect Core specifications support multiple authentication methods, but itsme® only supports <i>"private_key_jwt"</i> : it requires that each party exposes its public keys as a simple JWK Set document on a URI accessible to all, and keep its private set for itself. For itsme®, this URI can be retrieved from the [itsme® Discovery document](#OpenIDConfig), using the <i>"jwks_uri"</i> key.

Your private and public keys can be generated via Yeoman. To get started, install Yeoman and generator-itsme with NPM:

<code style=display:block;white-space:pre-wrap>$ npm install -g yo generator-itsme</code>

After installation, run the generator:

<code style=display:block;white-space:pre-wrap>$ yo itsme</code>

<aside class="notice">Don't forget to send your JWK Set URI by email to onboarding@itsme.be and we’ll make sure to complete the configuration for you in no time!
</aside>


<a name="AuthNRequest"></a>
## 2.4. Forging an Authentication Request

First, you will forg a HTTPS GET request that MUST be sent to the itsme® Authorization Endpoint. The itsme® Authorization Endpoint can be retrieved from the [itsme® Discovery document](#OpenIDConfig), using the key <i>"authorization_endpoint"</i>.

<aside class="notice">By opposition to the OpenID Connect specifications, POST method is not authorized when triggering the itsme® App through the Universal/App Link mechanism only support the HTTP GET method on the Authorisation Endpoint. More information about Universal links and App links can be found in the Appendix.
</aside>

The OpenID Connect Core specification defines a number of parameters to integrate in the HTTPS GET query string:

Parameter | Required | Description
:-------- | :--------| :----- 
**client_id** | Required |This is the client ID you received when registering your project in the [itsme® B2B portal](#Onboarding).
**response_type** | Required | This defines the processing flow to be used when forming the response. Because itsme® uses the Authorization Code Flow as described above, this value MUST be <i>"code"</i>.
**scope** | Required | The scope parameter allows the application to express the desired scope of the access request. It MUST contain the value <i>"openid"</i> and <i>"service:TEST_code"</i>, by replacing "TEST_code" with the service code you received when registering your project in the [itsme® B2B portal](#Onboarding).<br>You MAY also specify additional scopes, separated by spaces, to request more information about the User. Depending on your use case, the following additional scopes MAY apply:<ul><li>profile: will request the claims representing basic profile information. These are <i>"family_name"</i>, <i>"given_name"</i>, <i>"name"</i>, <i>"gender"</i>, <i>"birthdate"</i> and <i>"locale"</i>.</li><li>email: will request the <i>"email"</i> and <i>"email_verified"</i> claims.</li><li>phone: will request the <i>"phone_number"</i> and <i>"phone_number_verified"</i> claims.</li><li>address: will request the <i>"street_address"</i>, <i>"locality"</i>, <i>"postal_code"</i> claims.</li></ul>For more information on User attributes or claims, please consult the [ID claims](#Data) section.</br><br>An HTTP ERROR <i>"not_implemented"</i> will be returned if the required values are not specified.</br><br>Unrecognised values will be ignored.</br>
**redirect_uri** | Required | This is the URI to which the authentication response should be sent. This MUST exactly match the redirect URI, GET parameters included, of the specified service defined when registering your project in the [itsme® B2B portal](#Onboarding). Please note in Sandbox environment, the redirect_uri is NOT checked. This allows you to test your integration locally.
**request_uri** | Optional | This parameter enables OpenID Connect parameters to be passed by reference. The <i>"request_uri"</i> value is a URL using the https scheme referencing a resource containing a Request Object value, which is a JWT containing the request parameters. The URL MUST be shared with us when registering your project in the [itsme® B2B portal](#Onboarding).<br>See <a href="#RequestUri">Using request_uri parameter</a> for more details.</br>

On top of this, OpenID Connect specify that additionnal parameters MAY be added to the Authentication Request as a JWT Payload in the <i>"request_uri"</i> parameter:

Parameter | Required | Description
:-------- | :--------| :----- 
**state** | Strongly RECOMMENDED | An opaque value used in the Authentication Request, which will be returned unchanged in the Authorization Code. This parameter SHOULD be used for preventing cross-site request forgery (XRSF). <br>When deciding how to implement this, one suggestion is to use a private key together with some easily verifiable variables, for example, your client ID and a session cookie, to compute a hashed value. This will result in a byte value that will be infeasibility difficult to guess without the private key. After computing such an HMAC, base-64 encode it and pass it to the Authorization  Server as <i>"state"</i> parameter. Another suggestion is to hash the current date and time. This requires your application to save the time of transmission in order to verify it or to allow a sliding period of validity.</br>
**nonce** | Strongly RECOMMENDED | A string value used to associate a session with an ID Token, and to mitigate replay attacks. The value is passed through unmodified from the Authentication Request to the ID Token. Sufficient entropy MUST be present in the <i>"nonce"</i> values used to prevent attackers from guessing values. See <a href="http://openid.net/specs/openid-connect-core-1_0.html#NonceNotes" target="blank">OpenID Connect Core specifications</a> for more information.
**display** | Optional | ASCII string value that specifies how the Authorization Server displays the authentication and consent User interface pages to the User. MUST be <i>"page"</i> if provided.<br>Other values will yield an HTTP ERROR <i>"not_implemented"</i>.</br>
**prompt** | Optional | Space delimited, case sensitive list of ASCII string values that specifies whether the Authorization Server prompts the User for reauthentication and consent. MUST be <i>"consent"</i> if provided. 
**ui_locales** | Optional | User's preferred languages and scripts for the User interface (e.g.: OpenID web page). Supported values are: <i>"fr"</i>, <i>"nl"</i>, <i>"en"</i> and <i>"de"</i>. Any other value will be ignored.
**max_age** | Not supported | Any supplied value will be ignored.<br>As itsme® does not maintain a session mechanism, an active authentication is always required.</br>
<a name="acrvalues">**acr_values**</a> | Optional | Space-separated string that specifies the acr values that the Authorization Server is being requested to use for processing this Authentication Request, with the values appearing in order of preference.<br>2 values are supported:<ul><li>Basic level - let the User to choose either fingerprint usage (if device is compatible) or PIN<br><i>"tag:sixdots.be,2016-06:acr_basic"</i></br></li><li>Advanced level - force the User to use PIN<br><i>"tag:sixdots.be,2016-06:acr_advanced"</i></br></li></ul>When multiple values are provided only the most constraining will be used (advanced > basic). If not provided basic level will be used.</br><br>More information on security levels and context data can be found in the [Appendixes](#SecurityLevels).</br>
**claims** | Optional | This parameter is used to request specific claims. The value is a JSON object listing the requested claims. <br>See [User Data](#Data) for more information.</br>
**response_mode** | Not supported | Any supplied value will be ignored.
**id\_token\_hint** | Not supported | Any supplied value will be ignored.
**claims_locales** | Not supported | Any supplied value will be ignored.
**registration** | Not supported | Any supplied value will be ignored.

<aside class="notice">The JWT Payload in the <i>"request_uri"</i> parameter MAY be <b>signed</b> using the <a href="https://belgianmobileid.github.io/slate/jose.html" target="blank">JSON Web Signature</a> (JWS). It MAY also be <b>encrypted</b> using <a href="https://belgianmobileid.github.io/slate/jose.html" target="blank">JSON Web Encryption</a> (JWE). In this case, it MUST be signed then encrypted, with the result being a Nested JWT, as defined in the <a href="https://belgianmobileid.github.io/slate/jose.html" target="blank">JSON Web Token</a> (JWT) section.
</aside>

The following is a non-normative example of a request that would be sent to the Authorization Server:

<code style=display:block;white-space:pre-wrap>Authentication Request:<br></br>
    GET /oidc/authorization HTTP/1.1
    &response_type=code
    &client_id=MY_PARTNER_CODE
    &scope=openid service:TEST_code profile email
    &redirect_uri=https://test.istme.be
    &request_uri=https://test.istme.be:443/p/test<br></br>
Raw Request Object (not signed, not encrypted):<br></br>
    {
      "aud": "https://merchant.itsme.be/oidc/authorization",
      "scope": "openid service:TEST_code profile email",
      "redirect_uri": "https://test.istme.be",
      "response_type":"code",
      "client_id":"MY_PARTNER_CODE",
      "acr_values":"tag:sixdots.be,2016-06:acr_advanced",
      "iss":"MY_PARTNER_CODE",
      "nonce":"A_VALID_NONCE",
      "state":"A_VALID_STATE",
      "claims":{
        "userinfo":{
          "tag:sixdots.be,2016-06:claim_eid":null,
          "tag:sixdots.be,2016-06:claim_city_of_birth":null
          }
        }
     }</code>

<a name="AuthNResponse"></a>
## 2.5. Capturing an Authorization Code

### Capturing a successful Authorization Code

If the User is successfully authenticated and authorizes access to the data requested, itsme® will return an Authorization Code to your server component. This is achieved by returning an Authentication Response, which is a HTTP 302 redirect request to the <i>"redirect_uri"</i> specified previously in the Authentication Request. The following is a non-normative example of a successful Authentication Response:
 
<code style=display:block;white-space:pre-wrap>HTTP/1.1 302 Found
Location: https://client.example.org/cb?
code=SplxlOBeZQQYbYS6WxSbIA&
state=af0ifjsldkj</code>

The response will contain:

Values | Returned | Description
:----- |:-------- |:---
**code** | Always |The <i>"code"</i> parameter holds the Authorization Code which is a string value. The content of Authorization Code is opaque for you. This code has a lifetime of 3 minutes.
**state** | If requested |The <i>"state"</i> parameter will be returned if you provided a value in the Authentication Request. You should validate that the value returned matches the one supplied in the Authentication Request. The state value can additionally be used to mitigate against XSRF attacks by cryptographically binding the value of this parameter with a browser cookie.<br>If a wrong/unknown <i>"state"</i> is received, you should take it into account and refuse the related <i>"code"</i> or from a security detection/prevention point of view, monitor if it is a recurring pattern or not.</br>

 
### Handling Authentication Error Response

If the request fails due to a missing, invalid, or mismatching redirection URI, or if the client identifier is missing or invalid, the Authorization Server SHOULD inform the User of the error and MUST NOT automatically redirect him to the invalid redirection URI. 

If the User denies the Authentication Request or if the request fails for reasons other than a missing or invalid redirection URI, itsme® will return an error response to your application. As for a successful response this is achieved by returning a HTTPS 302 redirect request to the redirection_uri specified in the Authentication Request. The error response parameters are the following:

Values |	Returned | Description
:--|:--|:--
**error**	| Always |	Error type. 
**error_description** |	Optional	| Indicating the nature of the error.
**state** |	If requested	| Set to the value defined in the Authorisation Request, if any.

The following is a non-normative example of an error response:

<code style=display:block;white-space:pre-wrap>HTTP/1.1 302 Found
Location: https://client.example.org/cb?
error=invalid_request
&error_description=Unsupported%20response_type%20value
&state=af0ifjsldkj</code>

The following table describes the various error codes that can be returned in the <i>"error"</i> parameter of the error response:

Error | Description
:-- | :-- 
**interaction_required**  | The Authorization Server requires User interaction of some form to proceed.
**invalid_request_object** | The <i>"request"</i> parameter contains an invalid Request Object.
**registration_not_supported** | This error is returned because itsme® does not support use of the <i>"registration"</i> parameter.

All other HTTPS errors unrelated to OpenID Connect Core will be returned to the User using the appropriate HTTPS status code.

<a name="UniversalLinks"></a> 
## 2.6. Supporting Universal Links and App Links mechanism
Regardless of the application you are building you should make sure that your redirect URIs support the <a href="https://developer.apple.com/ios/universal-links/" target="blank">Universal links</a> and <a href="https://developer.android.com/studio/write/app-link-indexing" target="blank">App links</a> mechanism. Functionally, it will allow you to have only one single link that will either open your desktop web application, your mobile app or your mobile site on the User’s device.

Universal links and App links are standard web links (http://mydomain.com) that point to both a web page and a piece of content inside an app. When a Universal Link is opened, the app OS checks to see if any installed app is registered for that domain. If so, the app is launched immediately without ever loading the web page. If not, the web URL is loaded into the webbrowser.

An App link is the Android version of the Universal link.

How do Universal Links work in iOS and Android ? Before Universal Links, the primary mechanism to open up an app when it was installed was by trying to redirect to an app’s URI scheme  in the web browser. But there was no way to check if the app was installed or not. This meant that developers would try to call the URI scheme 100% of the time, in the off chance that the app was installed, then fallback gracefully to the App Store or Google Play Store when not by using a timer.

iOS Universal Links and Android App Links were intended to fix this. Instead of opening up the web browser first when a link is clicked, the OS will check if a Universal Link has been registered (a file should be there in the domain which contains the bundle id of the app and the paths the app should open) for the domain associated with the link, then check if the corresponding app is installed. If the app is currently installed, it will be opened. If it’s not, the web browser will open and the HTTPS link will load.

The specifications for the implementation of Universal links and App links can be found in the [Appendix](#Appendixes).

## 2.7. Exchanging the Authorization Code 
<a name="tokenEndpoint"></a> 

Once your server component has received an [Authorization Code](#AuthNResponse), your server can exchange it for an Access Token and an ID Token.

<aside class="notice">You might also read in the OpenID Connect Core specification about the Refresh Token, but we don't support them (we don't implement any session mechanism).</aside>

Your server makes this exchange by sending an HTTPS POST request to the itsme® Token Endpoint URI. This URI can be retrieved from the [itsme® Discovery document](#OpenIDConfig), using the key <i>"token_endpoint"</i>.

<aside class="notice">An Authorization Code can only be exchanged once. Attempting to re-exchange a code will generate a bad request response, outlined below in the section <a href="https://belgianmobileid.github.io/slate/login.html#3-6-managing-id-token-response" target="blank">Handling token error response</a>.</aside>

The request MUST include the following parameters in the POST body:

Parameter | Required | Description
:-- | :-- | :--
**grant_type** | Required | This MUST be set to <i>"authorization_code"</i>.
**code** | Required | The Authorization Code received in response to the Authentication Request.
**redirect_uri** | Required | The redirection URI supplied in the original Authentication Request. This is the URL to which you want the User to be redirected after the authorization is complete.
**client_assertion** | Required | To ensure that the request is genuine and that the tokens are not returned to a third party, you will be authenticated when making the Token Request.<br>The OpenID Connect Core specifications support multiple authentication methods, but itsme® only supports <i>"private_key_jwt"</i>. The JWT MUST be sent as the value of the <i>"client_assertion"</i> parameter.</br><br>See the <a href="https://belgianmobileid.github.io/slate/jose.html" target="blank">JOSE</a> specifications for more information.</br>
**client\_assertion\_type** | Required | This MUST be set to <i>"urn:ietf:params:oauth:client-assertion-type:jwt-bearer"</i>. 

<aside class="notice">The JWT Payload in the <i>"client_assertion"</i> parameter MUST be <b>signed then encrypted</b>, with the result being a Nested JWT, as defined in the <a href="https://belgianmobileid.github.io/slate/jose.html" target="blank">JSON Web Token</a> (JWT) section.
</aside>

The following is a non-normative example of a request to obtain an ID Token and Access Token:

<code style=display:block;white-space:pre-wrap>POST /token HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded
grant_type=authorization_code
&code=i1WsRn1uB1
&redirect_uri=https://test.istme.be
&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer
&client_assertion=PHNhbWxwOl ... ZT</code>

According to the <i>"private_key_jwt"</i> client authentication method, the <i>"client_assertion"</i> contains the following parameters in the JWT Payload:

Parameter | Required | Description
:-- | :-- | :-- 
**iss** | Required | The issuer of the <i>"private_key_jwt"</i>. This MUST contain the <i>"client_id"</i>. This is the client identifier (e.g. : Project ID) you received when registering your project in the [itsme® B2B portal](#Onboarding).
**sub** | Required | The subject of the <i>"private_key_jwt"</i>. This MUST contain the <i>"client_id"</i>. This is the client identifier (e.g. : Project ID) you received when registering your project in the [itsme® B2B portal](#Onboarding).
**aud** | Required | Value that identifies the Authorization Server as an intended audience. This MUST be the itsme® Token Endpoint URL: <i>"https://merchant.itsme.be/oidc/token"</i>.
**jti** | Required | The <i>"jti"</i> (JWT ID) claim provides a unique identifier for the JWT. The identifier value MUST be assigned by the you in a manner that ensures that there is a negligible probability that the same value will be accidentally assigned to a different data object; if the application uses multiple issuers, collisions MUST be prevented among values produced by different issuers as well.  The <i>"jti"</i> claim can be used  to prevent the JWT from being replayed. The <i>"jti"</i> value is a case-sensitive string. 
**exp** | Required | The <i>"exp"</i> (expiration time) claim identifies the expiration time on or after which the JWT MUST NOT be accepted for processing.  The processing of the <i>"exp"</i> claim requires that the current date/time MUST be before the expiration date/time listed in the <i>"exp"</i> claim. Implementers MAY provide for some small leeway, usually no more than a few minutes, to account for clock skew.  Its value is a JSON number representing the number of seconds from 1970-01-01T0:0:0Z as measured in UTC until the date/time.

<a name="TokenResponse"></a>
## 2.8. Managing Token Response

### Extracting a successful Token Response

If the Token Request has been sucessfully validated we will return an HTTP 200 OK response including ID and Access Tokens as in the example below:

<code style=display:block;white-space:pre-wrap>HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store
Pragma: no-cache
{
  "access_token": "SlAV32hkKG",
  "token_type": "Bearer",
  "expires_in": 3600,
  "id_token": "eyJhbGciOiJSUzI1NiIsImtpZCI6IjFlOWdkazcifQ.ewogImlzc
  yI6ICJodHRwOi8vc2VydmVyLmV4YW1wbGUuY29tIiwKICJzdWIiOiAiMjQ4Mjg5
  NzYxMDAxIiwKICJhdWQiOiAiczZCaGRSa3F0MyIsCiAibm9uY2UiOiAibi0wUzZ
  fV3pBMk1qIiwKICJleHAiOiAxMzExMjgxOTcwLAogImlhdCI6IDEzMTEyODA5Nz
  AKfQ.ggW8hZ1EuVLuxNuuIJKX_V8a_OMXzR0EHR9R6jgdqrOOF4daGU96Sr_P6q
  Jp6IcmD3HP99Obi1PRs-cwh3LO-p146waJ8IhehcwL7F09JdijmBqkvPeB2T9CJ
  NqeGpe-gccMg4vfKjkM8FcGvnzZUN4_KSP0aAp1tOJ1zZwgjxqGByKHiOtX7Tpd
  QyHE5lcMiKPXfEIQILVq0pc_E2DzL7emopWoaoZTF_m0_N0YzFC6g6EJbOEoRoS
  K5hoDalrcvRYLSrQAZZKflyuVCyixEoV9GfNQC3_osjzw2PAithfubEEBLuVVk4
  XUVrWOLrLl0nx7RkKU8NXNHq-rvKMzqg"
}</code>

The response body will include the following values:

Values | Returned | Description
:-- | :-- | :--
**access_token** | Always | The Access Token which may be used to access the userInfo Endpoint.
**token_type** | Always | Set to <i>"Bearer"</i>.
**id_token** | Always | The ID Token is a JSON Web Token (JWT) that contains User profile information (like the User's name, email, and so forth), represented in the form of claims. 
**at_hash** | Not supported | itsme® does not provide any value for this parameter.
**refresh_token** | Not supported | itsme® does not provide any value for this parameter as it only maintains short-lived session to enforce re-authentication.

With the following values returned in the <i>"id_token"</i>: 

Values |	Returned |	Description
:-- | :-- | :--
**iss**	| Always | Identifier of the issuer of the ID Token.
**sub** |	Always | An identifier for the User, unique among all itsme® accounts and never reused. Use <i>"sub"</i> in the application as the unique-identifier key for the User. It has 36 characters.
**aud**	| Always |	Audience of the ID Token. This will contain the <i>"client_id"</i>. This is the client identifier (e.g. : Project ID) you received when registering your project in the [itsme® B2B portal](#Onboarding).
**exp**	| Always |	Expiration time on or after which the ID Token MUST NOT be accepted for processing.
**iat** |	Always	| The time the ID Token was issued, represented in Unix time (integer seconds).
**auth_time** | Always | The time the User authentication occurred, represented in Unix time (integer seconds). 
**nonce** | If requested | String value used to associate a session with an ID Token, and to mitigate replay attacks. The value is passed through unmodified from the Authentication Request to the ID Token. Sufficient entropy MUST be present in the <i>"nonce"</i> values used to prevent attackers from guessing values. See <a href="http://openid.net/specs/openid-connect-core-1_0.html#NonceNotes" target="blank">the OpenID Connect Core specifications</a> for more information.
**acr** | If requested | Could be <i>"tag:sixdots.be,2016-06:acr_basic"</i> or <i>"tag:sixdots.be,2016-06:acr_advanced"</i> depending on the value entered in the <i>"acr_values"</i>  Authorisation Request parameter. If no value is provided in the <i>"acr_values"</i> parameter, the <i>"acr"</i> will contain <i>"tag:sixdots.be,2016-06:acr_basic"</i>.
**amr** | Never |
**azp** | Never |

However, before being able to store and use the returned values from <i>"id_token"</i>, you will first need to validate it by following the the ID Token validation rules described in the section below.

### ID Token validation

You MUST validate the ID Token in the Token Response in the following manner:

<ol>
  <li>As the ID Token is a Nested JWT object, you will have to decrypt and verify it using the keys and algorithms that you specified when registering your project in the <a href="#Onboarding" target="blank">itsme® B2B portal</a>. The process of decryption and signature validation is described in on.<br>If the ID Token is not encrypted, you SHOULD reject it.</br></li>
  <li>The Issuer identifier for itsme® (which is obtained by using the key <i>"issuer"</i> in the <a href="#OpenIDConfig" target="blank">itsme® Discovery document</a>) MUST exactly match the value of the <i>"iss"</i> claim.</li>
  <li>You MUST validate that the <i>"aud"</i> claim contains your <i>"client_id"</i> value registered in the <a href="#Onboarding" target="blank">itsme® B2B portal</a>. The ID Token MUST be rejected if the ID Token does not list the <i>"client_id"</i> as a valid audience.</li>
  <li>The current time MUST be before the time represented by the <i>"exp"</i> claim.</li>
</ol>

If all the above verifications are successful, you can use the subject (<i>"sub"</i>) of the ID Token as the unique identifier of the corresponding User.

### Handling token error response 

If the Token Request is invalid or unauthorized an HTTP 400 response will be returned as in the example:

<code style=display:block;white-space:pre-wrap>HTTP/1.1 400 Bad Request
Content-Type: application/json
Cache-Control: no-store
Pragma: no-cache
{
  "error": "invalid_request"
}</code>

The response will contain an error parameter and optionally <i>"error_description"</i> parameter.


<a name="Data"></a>
## 2.9. Obtaining User attributes or claims

### Creating the userInfo Request 

OpenID Connect Core specifications also allow your application to obtain basic profile information about a specific User in a interoperable way. This is achieved by sending a HTTPS GET request to the itsme® userInfo Endpoint, passing the Access Token value in the Authorization header using the Bearer authentication scheme. The itsme userInfo Endpoint URI can be retrieved from the [itsme® Discovery document](#OpenIDConfig), using the key <i>"userinfo_endpoint"</i>.

<code style=display:block;white-space:pre-wrap>GET https://merchant.itsme.be/oidc/userinfo HTTP/1.1
Authorization: Bearer <access token></code>

### Managing the userInfo Response 

The itsme® userInfo Endpoint will return a HTTP 200 OK response and the User claims in a Nested JWT format. The following is a non-normative example of a UserInfo Response:

<code style=display:block;white-space:pre-wrap>HTTP/1.1 200 OK
  Content-Type: application/json
  {
   "sub": "2i9rk3cg6mkt3v1dn9a1akwr6u4kp1zvj2oo",
   "name": "Jane Doe",
   "given_name": "Jane",
   "family_name": "Doe",
   "email": "janedoe@example.com",
   "picture": "[base64 encoded picture]"
  }</code>

<aside class="notice">For privacy reasons itsme® may elect to not return values for some requested claims. In that case the claim will be omitted from the JSON object rather than being present with a null or empty string value.</aside>

However, before being able to consume the claims from the userInfo response, you will first need to validate it by following the userInfo response validation rules described in the section below.

### UserInfo response validation

You MUST validate the userInfo reponse in the following manner:

<ol>
  <li>As the userInfo response is a Nested JWT object, you will have to decrypt and verify it using the keys and algorithms that the you specified when registering your project in the [itsme® B2B portal](#Onboarding). The process of decryption and signature validation is described in <a href="https://belgianmobileid.github.io/slate/jose#4-3-decrypting" target="blank">section 4.3</a> of the JOSE specifications.<br>If the userInfo response is not encrypted, the you SHOULD reject it.</br></li>
  <li>The <i>"sub"</i> claim will always be included in the response and this should be verified by you to mitigate against token substitution attacks. The <i>"sub"</i> claim in the userInfo response MUST be verified to exactly match the <i>"sub"</i> claim in the <i>"id_token"</i>; if they do not match, the userInfo response values MUST NOT be used.</li>
</ol>

When an error condition occurs an error response as defined in the <a href="https://tools.ietf.org/html/rfc6750" target="blank">OAuth 2.0 Bearer Token Usage specification</a> will be returned.

###  Capturing claims from the 'scope' parameter

On top of the <i>"openid"</i> and <i>"service:TEST_code"</i> values specified in the Authentication Request, you MAY also ask for additional scopes, separated by spaces, to request more information about the User. The following additional scopes MAY apply:

Parameter | Description
:-- | :--
**profile** | It will request the claims representing basic profile information. These are <i>"family_name"</i>, <i>"given_name"</i>, <i>"name"</i>, <i>"gender"</i>, <i>"birthdate"</i> and <i>"locale"</i>.
**email** | It will request the <i>"email"</i> and <i>"email_verified"</i> claims.
**phone** | It will request the <i>"phone_number"</i> and <i>"phone_number_verified"</i> claims
**address** | It will request the <i>"street_address"</i>, <i>"locality"</i>, <i>"postal_code"</i> claims.

The values returned via the itsme® userInfo Endpoint are those below:

Values |	Returned when request |	Description
:--- | :--- | :---
**family_name** | SHALL always | Returns user's surname(s) or last name(s). Note that in some cultures, people can have multiple family names or no family name; all can be present, with the names being separated by space characters.
**given_name** | MAY NOT | Returns user's given name(s) or first name(s). Note that in some cultures, people can have multiple given names; all can be present, with the names being separated by space characters.
**name** | SHALL always | Returns user's full name in displayable form including all name parts, possibly including titles and suffixes.
**gender** | SHALL always | Returns user's gender. Possible values are : 'female', 'male'
**birthdate** | MAY NOT | Return user's birthday, represented as a string in YYYY-MM-DD date format. This value is derived from the birthdate_as_string claim.
**locale** | MAY NOT | Returns user's IDR language, represented as a string format. Possible values are : 'en' 'fr' 'nl' 'de'
**email** | MAY NOT | Returns user's email address.
**email_verified** | MAY NOT | Returns 'true' if the user's e-mail address is verified; otherwise 'false'.
**phone_number** | SHALL always | Returns user's phone number, represented as a string format. For example : [+][country code] [subscriber number including area code]
**phone_number_verified** | SHALL always | Returns true if the user's e-phone number is verified; otherwise false.
**address** | SHALL always | As member of address JSON object
**street_address** | SHALL always | As member of address JSON object
**locality** | SHALL always | As member of address JSON object
**postal_code** | SHALL always | As member of address JSON object

###  Capturing claims from the 'claims' parameter

Typically, the values returned via the <i>"scope"</i> parameter only contain claims about the identity of the User. More information about the User MAY be requested by including additional parameters in the <i>"claims"</i> parameter as specified below:

Parameter | Description
:-- | :-- 
**tag:sixdots.be,2016-06:claim_nationality** | It will request the <i>"nationality"</i> claim.
**tag:sixdots.be,2016-06:claim_city_of_birth** | It will request the <i>"place of Birth - city"</i>, <i>"place of Birth - country"</i> claim.
**tag:sixdots.be,2016-06:claim_eid**  | It will request the <i>"eid"</i>, <i>"issuance_locality"</i>, <i>"validity_from"</i>, <i>"validity_to"</i>, <i>"read_date"</i> and <i>"national_number"</i> claims.
**tag:sixdots.be,2017-05:claim_device** | It will request the <i>"os"</i>, <i>"appName"</i>, <i>"appRelease"</i>, <i>"deviceLabel"</i>, <i>"debugEnabled"</i>, <i>"deviceID"</i>, <i>"osRelease"</i>, <i>"manufacturer"</i>, <i>"hasSimEnabled"</i>, <i>"deviceLockLevel"</i>, <i>"smsEnabled"</i>, <i>"rooted"</i>, <i>"imei"</i>, <i>"deviceModel"</i> and <i>"sdkRelease"</i> claims.
**tag:sixdots.be,2017-05:claim_photo** | It will request the <i>"e-ID Picture"</i> claim.
**tag:sixdots.be,2020-03:claim_birthdate_as_string** | Returns user's birthday. It is considered as official or at least coming unprocessed from the ID document.

The values returned via the itsme® userInfo Endpoint are those below:

<a name id="SecurityDataElements"></a>

Values | Returned when requested | Description
:-- | :-- | :-- 
**nationality** | SHALL always | Returns user's nationality, represented as a string for Belgian ID documents.
**place of Birth - city** | MAY NOT | Returns user's place of birth, represented as a string.
**place of Birth - country** | MAY NOT | Returns user's place of birth, represented as a string.
**eid** | SHALL always | Returns user's Belgian ID document information. 
**issuance_locality**  | MAY NOT | Returns user's Belgian ID document issuance locality, represented as a string.
**validity_from** | MAY NOT | Returns user's Belgian ID document issuance date, represented as a string in YYYY-MM-DDThh:mm:ss date format specified by ISO 8601.
**validity_to** | MAY NOT | Returns user's Belgian ID document expiry date, represented as a string in YYYY-MM-DDThh:mm:ss date format specified by ISO 8601.
**read_date** | MAY NOT | Returns user's Belgian ID card data extraction date, represented as a string in YYYY-MM-DDThh:mm:ss date format specified by ISO 8601.
**national_number** | SHALL always | Returns user's Belgian unique identification number, represented as a string with 11 digits in the form YY.MM.DD-xxx.cd where YY.MM.DD is the birthdate of the person, xxx a sequential number (odd for males and even for females) and cd a check-digit. Some exceptions could apply.
**os** | MAY NOT | The device operating system. The returned values will be <i>"ANDROID"</i> or <i>"iOS"</i>
**appName** | MAY NOT | The application name.
**appRelease** | MAY NOT | The application current release.
**deviceLabel** | MAY NOT | The name of the device. itsme(r) does not possess this information for every account.
**debugEnabled**  | MAY NOT | <i>"True"</i> if debug mode has been activated; otherwise <i>"false"</i>.
**deviceId** | MAY NOT | The device identifier.
**osRelease** | MAY NOT | The version of the OS running on your device.
**manufacturer** | MAY NOT | The brand of the device manufacturer.
**hasSimEnabled** | MAY NOT | It tells you if a SIM card is installed in the device, or not. The returned value is always <i>"true"</i> as long as itsme® can't be installed on tablets.
**deviceLockLevel** | MAY NOT | The type of action required to unlock the device (PIN, password, pattern, nothing).
**smsEnabled** | MAY NOT | True if can send SMS. On iOS, this means it’s an iPhone.
**rooted** | MAY NOT | The returned value is always <i>"false"</i> since itsme® can't be used on a jailbreaked/rooted device. 
**imei** | MAY NOT | The device IMEI value.
**deviceModel** | MAY NOT | The model of the device (e.g. iPhone 7)
**sdkRelease** | MAY NOT | The version of sdk on this device.
**e-ID Picture** | MAY NOT | Returns user's ID picture, represented as a string in Base64 image format. When decoded the image should have the following properties : 200 x 140px resolution, 24 BPP, JPEG. 
**birthdate_as_a_string** | MAY NOT | Returns user's birthday. It is considered as official or at least coming unprocessed from the ID document.


# 3. Mapping the User

To sign in successfully in your web desktop, mobile web or mobile application, a given user must be provisioned in OpenID Connect and then mapped to a user account in your database. By default, your application Server will use the subject identifier, or <i>"sub"</i> claim, in the ID Token to identify and verify a user account. Typically, the <i>"sub"</i> claim is a unique string that identifies a given user account. The benefit of using a <i>"sub"</i> claim is that it will not change, even if other user attributes (email, phone number, etc) associated with that account are updated. 

The <i>"sub"</i> claim value must be mapped to the corresponding user in your application Server. If you already mapped this <i>"sub"</i> to an account in your application repository, you should start an application session for that User.

If no user record is storing the <i>"sub"</i>  claim value, then you should allow the User to associate his new or existing account during the first sign-in session.

All these flows are depicted in the <a href="https://brand.belgianmobileid.be/document/39#/ux/ux-flows" target="blank">itsme® B2B portal</a>.

In a limited number of cases (e.g. technical issue,…) a user could ask itsme® to ‘delete’ his account. As a result the specific account will be ‘archived’ (for compliancy reasons) and thus also the unique identifier(s) (e.g. <i>"sub"</i>), used to interact with the different Service Providers the specific users is active with, will be automatically deleted in our database.

If the same user would opt to (re)create an itsme® afterwards, he will need to re-bind his itsme® account with your application server (as the initial identifier is no longer valid as explained before). To re-bind his itsme® account one of the above scenario should be used. After successful (re)binding you will need to overwrite the initial reference with the new ‘sub’ claim value in your database.

 
# 4. Appendixes
<a name="Appendix"></a> 

## 4.1. Universal Links on iOS

<code style=display:block;white-space:pre-wrap>{
  "applinks": {
    "apps": [],
    "details": [
      {
        "appID": “JHGFJHHYX.com.facebook.ios",
        "paths": [
          "*"
        ]
      }
    ]
  }
}</code>

Integration is going to be pretty straightforward, all details can be found in below steps (as documented on <a href="https://developer.apple.com/ios/universal-links/" target="blank">Universal Links official documentation</a>):

<ol>
  <li>Register your app at developer.apple.com.</li>
  <li>Enable ‘Associated Domains’ on your app identifier.</li>
  <li>Enable ‘Associated Domain’ on in your Xcode project.</li>
  <li>Add the proper domain entitlement and make sure the entitlements file is included at build: Xcode will do it automatically by itself.</li>
  <li>Create the ‘apple-app-site-association’ file (AASA). The AASA file contains a JSON object with a list of apps and the URL paths on the domain that should be included or excluded as Universal Links.<br>The JSON object will contain:</br></li>
</ol>

Parameter |	Description
:-- | :--
appID	| Built by combining your app’s Team ID (it should be retrieved from https://developer.apple.com/account/#/membership/) and the Bundle Identifier. In the example attached, JHGFJHHYX is the Team ID and com.facebook.ios is the Bundle ID.
paths	| Array of strings that specify which paths are included or excluded from association. Note: these strings are case sensitive and that query strings and fragment identifiers are ignored.

<ol>  
  <li value="6">Upload the ‘apple-app-site-association’ file to your HTTPS web server for the redirection URI communicated in the Authentication Request. The file can be placed at the root of your server or in the .well-known subdirectory.</li>
</ol>

<aside class="notice">While hosting the AASA file, please ensure that the AASA file:
    <ul>
      <li>is served over HTTPS.</li>
      <li>uses application/json MIME type.</li>
      <li>don’t append .json to the apple-app-site-association filename.</li>
      <li>has a size not exceeding 128 Kb (requirement in iOS 9.3.1 onwards)</li>
    </ul>  
</aside>

<ol>
  <li value="7">Check if the AASA file is valid and is accessible by using the <a href="https://branch.io/resources/aasa-validator/#resultsbox" target="blank">following link</a>.</li>
  <li>Add an entitlement to all redirect URI that the your app need to supports. In Xcode, open the Associated Domains section in the Capabilities tab and add an entry for each Redirect URI that your app supports, prefixed with `applinks`.<br>To match all subdomains of an associated redirect URI, you can specify a wildcard by prefixing `*.` before the beginning of a specific Redirect URI (the period is required). Redirect URI matching is based on the longest substring in the `applinks` entries. For example, if you specify the entries `applinks:*.mywebsite.com` and `applinks:*.users.mywebsite.com`, matching for the redirect URI `emily.users.mywebsite.com` is performed against the longer `*.users.mywebsite.com` entry. Note that an entry for `*.mywebsite.com` does not match `mywebsite.com` because of the period after the asterisk. To enable matching for both `*.mywebsite.com` and `mywebsite.com`, you need to provide a separate `applinks` entry for each.</br></li>
</ol>
  
<aside class="notice">Apple doc says to limit this list to no more than about 20 to 30 domains</aside>
 
<ol>
  <li value="9">Update the app delegate to respond appropriately when it receives the `NSUserActivity` object. After all above steps are completed perfectly, when the User click a universal link, the app will open up and the method <i>"application:continueUserActivity:restorationHandler"</i> will get called in <i>"Appdelegate"</i>. When iOS launches the the app after a User taps a universal link, you receive an <i>"NSUserActivity"</i> object with an <i>"activityType"</i> value of <i>"NSUserActivityTypeBrowsingWeb"</i>. The activity object’s <i>"webpageURL"</i> property contains the redirect URI that the user is accessing. The webpage URL property always contains an HTTPS URL, and you can use <i>"NSURLComponents"</i> APIs to manipulate the components of the URL.<br>For getting the URL parameters, use the function aside.</br><br>Also if you want to check if the app had opened by clicking a universal link or not in the `didFinishLaunchingWithOptions` method below.</br>
  </li>
</ol>

<code style=display:block;white-space:pre-wrap>func application(_ application: UIApplication, continue userActivity: NSUserActivity, restorationHandler: @escaping ([Any]?) -> Void) -> Bool {
    print("Continue User Activity called: ")
    if userActivity.activityType == NSUserActivityTypeBrowsingWeb {
        let url = userActivity.webpageURL!
        print(url.absoluteString)
        //handle url and open whatever page you want to open.
    }
    return true
}</code>

<code style=display:block;white-space:pre-wrap>//playground code..
var str = “https://google.com/contents/someotherpath?category=series&contentid=1562167825"
let url = URL(string: str)
func queryParameters(from url: URL) -> [String: String] {
let urlComponents = URLComponents(url: url, resolvingAgainstBaseURL: false)
var queryParams = [String: String]()
for queryItem: URLQueryItem in (urlComponents?.queryItems)! {
if queryItem.value == nil {
continue
}
queryParams[queryItem.name] = queryItem.value
}
return queryParams
}
// print the url parameters dictionary
print(queryParameters(from: url!))

//It will print [“category”: “series”, “contentid”: “1562167825”]</code>

<code style=display:block;white-space:pre-wrap>func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey : Any]? = nil) -> Bool {
 var isUniversalLinkClick: Bool = false
 if launchOptions[UIApplicationLaunchOptionsUserActivityDictionaryKey] {
 let activityDictionary = launchOptions[UIApplicationLaunchOptionsUserActivityDictionaryKey] as? [AnyHashable: Any] ?? [AnyHashable: Any]()
 let activity = activityDictionary[“UIApplicationLaunchOptionsUserActivityKey”] as? NSUserActivity ?? NSUserActivity()
 if activity != nil {
 isUniversalLinkClick = true
 }
 }
 if isUniversalLinkClick {
 // app opened via clicking a universal link.
 } else {
 // set the initial viewcontroller
 }
 return true
}</code>

## 4.2. App Links on Android

The App Links Assistant in Android Studio can help you create intent filters in your manifest and map existing URLs from your website to activities in your app. Follow below steps to configure the App links (as documented on <a href="https://developer.android.com/studio/write/app-link-indexing" target="blank">App Links official documentation</a>):

<ol>
  <li>Add the intent filters to your manifest. Go through the your manifest and select Tools &gt; App Links Assistant. Click Open URL Mapping Editor and then click Add  at the bottom of the URL Mapping list to add a new URL mapping.</li>
  <li>Add details for the new URL mapping:
    <ul>
      <li>Entering your redirect URI in the <i>"host"</i> field.</li>
      <li>Add a <i>"path"</i>, <i>"pathPrefix"</i>, or <i>"pathPattern"</i> for the redirect URIs you want to map. For example, if you have a recipe-sharing app, with all the recipes available in the same activity, and your corresponding website's recipes are all in the same <i>"/recipe directory"</i>, use <i>"pathPrefix"</i> and enter <i>"/recipe"</i>. This way, the redirect URI http://www.recipe-app.com/recipe/grilled-potato-salad maps to the activity you select in the following step.</li>
      <li>Select the Activity the redirect URI should take Users to.</li>
      <li>Click OK.</li>
    </ul>
  <li>The App Links Assistant adds intent filters based on your URL mapping to the <i>"AndroidManifest.xml"</i> file, and highlights it in the <i>"Preview"</i> field. If the you would like to make any changes, click Open <i>"AndroidManifest.xml"</i> to edit the intent filter.</li>
</ol>
 
<aside class="notice">To support more links without updating the app, you should define a URL mapping that supports future redirect URIs.</aside>

<ol>
  <li value="4">To verify the URL mapping works properly, enter a URL in the Check URL Mapping field and click Check Mapping. If it's working correctly, the success message shows that the URL entered maps to the activity you selected.</li>
  <li>Handle incoming links. Once you have verified that the URL mapping is working correctly, you MUST add the logic to handle the intent he created.
    <ul>
      <li>Click Select Activity from the App Links Assistant.</li>
      <li>Select an activity from the list and click Insert Code.</li>
    </ul>
    <br>The App Links Assistant adds code to the activity's Java file, similar to the one aside.</br>
    <br>However, this code isn't complete on its own. You MUST now take an action based on the URI in <appLinkData>, such as display the corresponding content. For example, for the recipe-sharing app, the code might look like the sample aside.</br>
  </li>
</ol>

<code style=display:block;white-space:pre-wrap>protected void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);
  ...
  handleIntent(getIntent());
}
protected void onNewIntent(Intent intent) {
  super.onNewIntent(intent);
  handleIntent(intent);
}
private void handleIntent(Intent intent) {
    String appLinkAction = intent.getAction();
    Uri appLinkData = intent.getData();
    if (Intent.ACTION_VIEW.equals(appLinkAction) && appLinkData != null){
        String recipeId = appLinkData.getLastPathSegment();
        Uri appData = Uri.parse("content://com.recipe_app/recipe/").buildUpon()
            .appendPath(recipeId).build();
        showRecipe(appData);
    }
}</code>

<ol>
  <li value="6">Associate the app with the redirect URI. After setting up URL support for your app, the App Links Assistant generates a Digital Asset Links file you can use to associate his website with your app. As an alternative to using the Digital Asset Links file, you can associate your site and app in Search Console. To associate the app and the website using the App Links Assistant, click Open the Digital Asset Links File Generator from the App Links Assistant:</li>
  <li>Enter your Site domain and Application ID.</li>
  <li>To include support in your Digital Asset Links file for Smart Lock for Passwords, select Support sharing credentials between the app and the website and enter your site's login URL. This adds the following string to your Digital Asset Links file declaring that your app and website share sign-in credentials: <i>"delegate_permission/common.get_login_creds"</i>.</li>
  <li>Specify the signing config or select a keystore file. Make sure to select the right config or keystore file for either the release build or debug build of your app. If you want to set up his production build, use the release config. If you want to test his build, use the debug config.</li>
  <li>Click <i>"Generate Digital Asset Links"</i> file.</li>
  <li>Once Android Studio generates the file, click <i>"Save file"</i> to download it.</li>
  <li>Upload the <i>"assetlinks.json"</i> file to redirect URI site, with read-access for everyone, at <i>"https://<yoursite>/.well-known/assetlinks.json"</i>.</li>
  <li>Click <i>"Link and Verify"</i> to confirm that you've uploaded the correct Digital Asset Links file to the correct location.
</ol>

<aside class="notice">The system verifies the Digital Asset Links file via the encrypted HTTPS protocol. Make sure that the assetlinks.json file is accessible over an HTTPS connection, regardless of whether your app's intent filter includes https.</aside>


<a name="SecurityLevels"></a>
## 4.3. Security levels and context data

Depending on the required level of assurance your application MUST ensure the correct ‘security level’ is requested. Two ‘security levels’ are supported – ‘basic’ and ‘advanced’. Opting for ‘advanced’ will result in an itsme® code entry being enforced to validate the specific transaction (even if the user has activated touch ID/fingerprint). With the ‘basic’ level also touch ID/fingerprint next to the itsme® code are supported to validate the specific transaction.

The following table defines the different possible scenarios:

Security level | Login | Share Data
:--- | :--- | :---
**basic** | Allowed | Allowed
**advanced** | Allowed | Allowed

Next to the security levels, your application MAY also request additional security data to determine the context in which the transactions have been executed. The security data elements, alone or in combination with each other, indicate a given risk exposure. By providing these elements, itsme® allows you to detect fraud attempts or effective fraudulent transactions. The security data elements that MAY be used to associate a risk exposure to a specific transaction are available in the section [Capturing claims from the userInfo Endpoint](#SecurityDataElements).


<a name="RequestUri"></a>
## 4.4. Using request_uri parameter

The <i>"request_uri"</i> parameter enables the Authentication Requests to be passed by reference, meaning that the Request Object value is retrieved from the resource at the specified URL.

When the <i>"request_uri"</i> parameter is used, the OpenID Connect request parameter values contained in the referenced JWT supersede those passed using the OAuth 2.0 request syntax. 

The following validations should be done when using the <i>"request_uri"</i> parameter:

<ol>
  <li>The values for the <i>"response_type"</i> and <i>"client_id"</i> parameters MUST be filled in the Authentication Request, since they are REQUIRED in the OpenID Connect Core specifications. The values for these parameters MUST match those in the Request Object, if present.</li>
  <li>Even if a <i>"scope"</i> parameter is present in the Request Object value, a <i>"scope"</i> parameter – containing the <i>"openid"</i> scope value to indicate to the underlying OpenID Connect Core logic that this is an OpenID Connect request – MUST always be passed in the Authentication Request.</li>
  <li>The Request Object MUST be <b>signed</b> using the <a href="https://belgianmobileid.github.io/slate/jose.html" target="blank">JSON Web Signature</a> (JWS). If signed, the Request Object SHOULD contain the claims <i>"iss"</i> (issuer) and <i>"aud"</i> (audience) as members. The <i>"iss"</i> value SHOULD be your Client ID. The <i>"aud"</i> value SHOULD be <i>"https://merchant.itsme.be/oidc/authorization"</i>.</li>
  <li>The Request Object MAY also be <b>encrypted</b> using <a href="https://belgianmobileid.github.io/slate/jose.html" target="blank">JSON Web Encryption</a> (JWE). In this case, it MUST be signed then encrypted, with the result being a Nested JWT, as defined in the <a href="https://belgianmobileid.github.io/slate/jose.html" target="blank">JSON Web Token</a> (JWT) section.</li>
  <li>You need to store the Request Object resource remotely at a URL the the Authorization Server can access. This URL is the Request URI, <i>"request_uri"</i>. Usage of 'localhost' is not permitted.
  <li>The Request URI MUST contain the port 443 as in this example: https://test.istme.be:443/p/test.</li>
  <li>The Request URI value is a URL using the <i>https</i> scheme.</li>
  <li>The Content-Type SHOULD be <i>"application"/jwt</i>
</ol>

Enclosed you will find a non-normative example of an Authorization Request using the <i>"request_uri"</i> parameter:

<code style=display:block;white-space:pre-wrap>Authentication Request:<br></br>
    GET /oidc/authorization HTTP/1.1
    &response_type=code
    &client_id=MY_PARTNER_CODE
    &scope=openid service:TEST_code profile email
    &redirect_uri=https://test.istme.be
    &request_uri=https://test.istme.be:443/p/test<br></br>
Raw Request Object (not signed, not encrypted):<br></br>
    {
      "aud": "https://merchant.itsme.be/oidc/authorization",
      "scope": "openid service:TEST_code profile email",
      "redirect_uri": "https://test.istme.be",
      "response_type":"code",
      "client_id":"MY_PARTNER_CODE",
      "acr_values":"tag:sixdots.be,2016-06:acr_advanced",
      "iss":"MY_PARTNER_CODE",
      "nonce":"A_VALID_NONCE",
      "state":"A_VALID_STATE",
      "claims":{
        "userinfo":{
          "tag:sixdots.be,2016-06:claim_eid":null,
          "tag:sixdots.be,2016-06:claim_city_of_birth":null
          }
        }
     }</code>
    
    
## 4.5 Supported character set

The character set we support for free text fields is ISO 8859-15. You can buy the specification on [ISO website](https://www.iso.org/standard/29505.html) or find a free version on [Wikipedia](https://en.wikipedia.org/wiki/ISO/IEC_8859-15#Codepage_layout). You might be interested in knowing that, although most usual characters are supported, some softwares-generated characters like curly apostrophes and long dashes are not part of ISO 8859-15. If you provide a non-supported character in a free text field the flow will be stopped and you will receive an error message back.





