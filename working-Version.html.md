 
 ---
 title: itsme(r) OpenID Connect documentation
 
 language_tabs: # must be one of https://git.io/vQNgJ
   - json: JSON
   - http: HTTP
 
 toc_footers:
  <!-- - <a href='#'>Sign Up for a Developer Key</a> -->
   - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>
 
 search: true
 ---
 # Who should read this document
 This document is intended to be read by Service Provider developers who are in charge of developing and integrating the itsme(r) services. In this document, we speak directly to the developers, using 'you' to designate them.
 
 # The itsme® services in a few words
 
 This documentation describes our OAuth 2.0 implementation of **itsme® Login** and **itsme(r) Approval**, which conforms to the [OpenID Connect 1.0](http://openid.net/specs/openid-connect-core-1_0.html) specifications.
 
 ## Login
 
 **itsme® Login** is a service provided by [Belgian Mobile ID](https://www.belgianmobileid.be) (BMID) to allow End-Users to login securely to your application. During your login flow, you will delegate the end user authentication to BMID.
 
 In order to let you identify the user authenticated, BMID uses pairwise user identifier, meaning each Partner will have a unique *User Code* for the same User. Doing so, nobody except BMID can link one given *User Code* of Partner to a specific User identity. 
 
 ## Confirm
 **itsme® Confirm** is a service provided by [Belgian Mobile ID](https://www.belgianmobileid.be) (BMID) to allow End-Users to give their consent to a specific action. You will use this service when you need to have a strong consent.  From regulation perspective, the consent obtained from the end user through **itsme(r) Confirm** has the same strength as a payment confirmation in a web banking interface using card readers. 
 
 ## Share Data
 
 If purpose of use is stated during the on-boarding process and consent is provided by the End-User during Authentication, end user data can be shared with your application.
 
 Data is currently shared only during Login or Approval. Off-line access to User information is not authorised.
 
 The following sets of Data are available:
 
 - **Verified Identity Data**: identity information retrieved from the National eID Card 
 - **Commercial Information**: information provided by the end-user 
 - **Security Information**: information retrieved during execution of the service that could impact security level of the transaction. 
 
 ### Verified Identity Data
 
 The user identity data provided by **itsme®** are Data coming from the National eID Card. These datas are provided to **itsme®** during user enrolment either directly from a card readout or indirectly through an Identity Registrar (IDR) having a strong identity verification process (e.g. face-to-face KYC with eID readout)  in-place.  
 
 Data | Definition
 -- | --
 **Full Name** | Full name is a concatenation of firstname, middlenames and lastname.
 **Date of birth** | Birthdate 
 **Place of birth** | Place of birth. ***Note**: this information can be localized* 
 **Gender** | Gender
 **Language** | Language
 **Nationality** | Nationality
 **Address** | Address containing street, street number, postal box, locality, postal code and country
 **Passport Number** | Passport Number
 **NRN** | National Registry Number
 **E-ID picture** | Picture taken from the National eID Card in low-resolution.
 **E-ID Metadata** | See [E-ID Metadata Information](#e-id-info)
 
 #### <a name="e-id-info"></a>E-ID Metadata Information
 Provides some information about the eID card readout related to the identity data provided by **itsme(r)**.
 
 Data | Definition
 -- | -- 
 **eID Serial Number** | the electronic ID card serial number.
 **issuance_locality** | the issuance locality.
 **Validity from** | eID card validity “from” date.
 **Validity to** | eID card validity “to” date.
 **Certificate Validity** | the certificate validity.
 **Read Date** | the data extraction date.
 
 ### Commercial Data
 
 Data | Definition
 -- | --
 **Phone Number** | Verified phone number associated to the **itsme(r)** user account.  
 **E-Mail Address** | E-Mail address. Not Verified. itsme(r) does not yet make use of email .
 
 ### Security Data
 
 Data | Definition
 -- | --
 **Device** | Information about the end user device. See [Device Information](#device-information)
 
 #### <a name="device-information"></a>Device Information 
 
 Data | Definition
 -- | -- 
 **OS** | the device operating system (supported values: {`ANDROID`, `IOS`})
 **Device Identifier** | Device identifier.
 **Application Name** | Application name.
 **Application Release** | Application current release.
 **Device Label** | Name of the device.
 **Debug Enabled** | True if debug mode is activated.
 **OS Release** | Version of the OS running on your Device.
 **Manufacturer** | Brand of the device manufacturer (‘Apple’ on iOS, device specific on Android). 
 **SIM Enabled** | True if there is a SIM in the Device. Should be always true, as long as BMID keeps forbidding installing **itsme(r)** on a tablet.
 **Lock Level** | The type of action to be performed to unlock the Device. On iOS : TOUCH_ID, PASSCODE or NONE if User protected his Device with TouchID, PIN or nothing.
 **SMS Enabled** | True if can send SMS. On iOS, this means it’s an iPhone. 
 **Rooted** | True if the device is jailbreaked/rooted.
 **IMEI** | Device IMEI value.
 **Model** | Model of the Device. e.g. SAMSUNG GALAXY A5
 **MSISDN** | User’s phone number. 
 **SDK Release** | SDK release 
 
 ## On-boarding Process
 
 Before your application can use **itsme®** for user login, you must set up a project to obtain OIDC credentials, set redirect URIs for your services, and customise the branding information that your users see on the **itsme®** user-consent screen.
 
 ### Customize the user consent screen
 SP provides 
 
 - Partner name, description and localized labels
 - Services' name, description and localized labels
 - Data access with justification (to comply with GDPR)
 
 ### Obtain OAuth 2.0 credentials
 BMID provides 
 
 - Partner Code, used as **client_id**
 - Service Code for each Service (see [Service Code concept](#ServiceCodes))

 ### Set a redirect URI and Certificates
 SP provides 
 
 - the redirect URIs for each Service to use within the OpenID Connect protocol to send back the response of the Authentication Request. 
 - the JWKSet HTTPS endpoint exposing the signing and encryption public certificates of the SP.
 - the SSL/TLS certificate used on the JWKSet HTTPS endpoint of the SP.
 
 ### itsme® OpenID Configuration
 The OpenID Connect protocol requires the use of multiple endpoints for authenticating users, and for requesting resources including tokens, user information, and public keys.
 
 To simplify implementations and increase flexibility, OpenID Connect allows the use of a "Discovery document", a JSON document found at a well-known location containing key-value pairs which provide details about the OpenID Connect provider's configuration including, 
- URIs of the authorization, 
- token, 
- userinfo, 
- supported claims 
-  public-keys endpoints to interact with it. 
 
 <aside class="success">The Discovery document for itsme® service may be retrieved from: <a href="https://merchant.itsme.be/oidc/.well-known/openid-configuration">https://merchant.itsme.be/oidc/.well-known/openid-configuration</a></aside>
 
 
 Field  names and meanings in this document are defined in [OpenID Connect Discovery 1.0](https://openid.net/specs/openid-connect-discovery-1_0.html)
 
  #### <a name="ServiceCodes"></a> Service Code Concept
 
To be able to use an itsme service (such as login, confirm, sign, share data) you should be provided a service instance for it. The service code is the identifier of this instance. The same Service Provider may utilise several service instances. 

For example, assuming that one SP would like to use login as an itsme(r) service for business and private channels. In this case, SP could ask BMID to allocate two service instances, one issued for private account login, one for business account login. Consent screen needs to be customised for each instance.
 
 # Authenticating the User
 
 **itsme(r) Login** is based on the [Authorisation Code Flow](http://openid.net/specs/openid-connect-core-1_0.html#CodeFlowAuth) of OpenID Connect 1.0.
  
 The Authorization Code Flow goes through the following steps as defined in  [http://openid.net/specs/openid-connect-core-1_0.html#CodeFlowSteps](http://openid.net/specs/openid-connect-core-1_0.html#CodeFlowSteps)
 
 ## 1. Authentication Request
 As per the OpenID Connect specification [http://openid.net/specs/openid-connect-core-1_0.html#AuthRequest](http://openid.net/specs/openid-connect-core-1_0.html#AuthRequest) and [http://openid.net/specs/openid-connect-core-1_0.html#AuthorizationEndpoint](http://openid.net/specs/openid-connect-core-1_0.html#AuthorizationEndpoint)
 
 The first step is forming an HTTPS request with the appropriate URI parameters. Please nNote the use of HTTPS rather than HTTP in all the steps of this process; HTTP connections are refused. You should retrieve the base URI from the [Discovery document](https://merchant.itsme.be/oidc/.well-known/openid-configuration) using the key **authorization_endpoint**. The following discussion assumes the endpoint is `https://merchant.itsme.be/oidc/authorize`.
 
 **itsme(r)** supports the use of the HTTP `GET` and `POST` methods. If using the HTTP `POST` method, the request parameters must be serialized using [Form Serialization](http://openid.net/specs/openid-connect-core-1_0.html#FormSerialization).
 
 ### Request Examples
 
 >Example of a minimal Authorization request
 
 ```http--inline
 GET /authorize?response_type=code
 &scope=openid%20profile%20email%20service%3Aclient.registration
 &client_id=s6BhdRkqt3
 &state=af0ifjsldkj
 &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb HTTP/1.1
 Host: server.itsme.be
 ```
 
 For a basic request, specify the following parameters:
 
 Parameter | Required | Comment | Parameter Explained
 --------- | ------| ------| ------------------
 **client_id** | Required | MUST be the Partner Code you obtained from BMID during on-boarding process and this value will be unique to each partner per environment. This information is in the on-boarding file provided by BMID.|
 **response_type** | Required | MUST be <code>code</code>.
 **scope** | Required | MUST contain at least `openid` or an HTTP ERROR `not_implemented` will be returned. `offline_access` value will yield an error. MUST also includes the target service in the form `service:<SERVICE_CODE>` as provided by BMID during the on-boarding process (see [Service Code concept](#ServiceCode) for further information). ***Note**: Requested data will only be provided based on your current accesses*. These accesses are specified in the on-boarding file provided by BMID.
 **redirect_uri** | Required | should be the HTTPS endpoint on your server that will receive the response from **itsme(r)**. This value MUST match one of the values provided to BMID during on-boarding process. ***Note**: the Partner can define different **redirect_uri** specific to each Service.*
 **state** | an appropriate value is RECOMMENDED | should include the value of the anti-forgery unique session token, as well as any other information needed to recover the context when the user returns to your application.
 **nonce** | an appropriate value is RECOMMENDED | a random value generated by your app that enables replay protection when present.| The purpose is to prevent id token to be displaced by an attacker by sending a valid id token from a different request in order to trick user visiting another website.
 **login_hint** | Optional | OPTIONAL and supported, though not recommended. Only phone numbers are supported as `login_hint`. Format is: `<coutrycode>+<phonenumber>`. E.g. `login_hint=32+123456789`. Usage of claim value `phone_number` in an encrypted request object is recommended in order to avoid disclosure of phone number of the enduser on the user agent (such as mobile app or web browser)
 **display** | Optional | MUST be `page` if provided. Other values will yield an HTTP ERROR `not_implemented`.
 **prompt** | Optional | MUST be `consent` if provided.
 **ui_locales** | Optional | Can be used to specify the language to be used by the OpenID login page. Supported languages are: `fr`, `nl`, `en` and `de`. Any other value will be ignored.
 **max_age** | Optional | Supported but not used: **itsme(r)** will always actively re-authenticate the End-User. 
 **acr_values** | Optional | OPTIONAL and supported, though not recommended. Possible values are tag:itsmetag:sixdots.be,2016-06:acr_basic, tag:itsmetag:sixdots.be,2016-06:acr_advanced. When multiple values are provided only the most constraining will be used (advanced > basic). If not provided basic level will be used. As there is no such idea of an existing session on itsme Core, even if the `acr_values` is requested as a voluntarily claim, the acr value returned will always be the more constraining method in the `acr_values` list, or the authentication will fail. Usage of acr parameter in the request object is recommended over this parameter as it will be signed in the JWT token/
 **claims** | Optional | Not recommended. Usage of claims parameter in the request object is recommended over this parameter as it will be signed in the JWT token, and the data will be encrypted
 **request** | Optional | See [Passing Request Parameters as JWTs](#JWTRequest)
 **response_mode** | Unsupported | MUST not be used. Any supplied value will be ignored.
 **id\_token\_hint** | Unsupported | Ignored if provided.
 **claims_locales** | Unsupported | None are supported.
 **request_uri** | Unsupported | Not supported (yet)
 **registration** | Unsupported | Not supported due to “client dynamic registration”is not supported. The client registration process is done during the partner onboarding.
 
 ### Authentication Response
 An Authentication Response is an [OAuth 2.0 Authorization Response](https://tools.ietf.org/html/rfc6749#section-4.1.2) message. As such, the Authentication Response will return the following parameters:
 
 Parameter | Provided | Description
 -- | -- | --
 **code** | Always | Authorization code to later provide to the token endpoint. This code has a lifetime of 3 minutes.
 **state** |  | The exact value received from the client, if the parameter was present in the Authentication Request.
 
 > Example of Authentication Response
 
 ```http--inline
 HTTP/1.1 302 Found
 Location: https://client.example.org/cb?
   code=SplxlOBeZQQYbYS6WxSbIA
   &state=af0ifjsldkj
 ```
 
 ### Errors
 As per http://openid.net/specs/openid-connect-core-1_0.html#AuthError
 
 If the authentication is NOT successful, the following errors can be triggered by itsme(r)
 
 Error | Description
 -- | --
 `interaction_required`  | The Authorization Server requires End-User interaction of some form to proceed.
 `invalid_request_object` | The request parameter contains an invalid Request Object.
 `request_uri_not_supported` | does not support use of the request_uri parameter.
 `registration_not_supported` | does not support use of the registration parameter.
 
 
 
 
 ## 2. Token Request
 
 As per the OpenID Connect specification http://openid.net/specs/openid-connect-core-1_0.html#TokenRequest
 
 The Authentication Response includes a `code` parameter, a one-time authorization code that your server can exchange for an ID token. Your server makes this exchange by sending an HTPS `POST` request. The `POST` request is sent to the token endpoint, which you should retrieve from the [Discovery document](https://merchant.itsme.be/oidc/.well-known/openid-configuration) using the key **token_endpoint**. The following discussion assumes the endpoint is `https://merchant.itsme.be/oidc/token`. Please note that BMID only supports `private_key_jwt` as client authentication method. The `client_secret` authentication methods are not supported since they are considered less secure. The Token Request must include the following parameters in the `POST` body:
 
 Parameter | Required | Comment
 -- | -- | --
 **grant_type** | Required | Must be `authorization_code`
 **code** | Required | The **code** value provided in the Authentication Response
 **redirect_uri** | Required | The **redirect_uri** used in the Authentication Request
 **client\_assertion\_type** | Required | Must be `urn:ietf:params:oauth:client-assertion-type:jwt-bearer` ***Note***: this specifies that the authentication method type is `private_key_jwt`, according to [Section 9](http://openid.net/specs/openid-connect-core-1_0.html#ClientAuthentication) of the OpenID specification.
 **client_assertion** | Required | Must be a valid JWT complying with the `private_key_jwt` client authentication method as defined in [Section 9](http://openid.net/specs/openid-connect-core-1_0.html#ClientAuthentication) of the OpenID specification. This JWT must be signed.
 
 According to the `private_key_jwt` client authentication method, the **client assertion** JWT must contain the following properties:
 
 Property | Comment
 -- | --
 **iss** | The issuer of the `private_key_jwt` (the client ID). MUST be the Partner Code you obtained from BMID during on-boarding process (this information is in the onboarding file provided by BMID). 
 **sub** | The subject of the `private_key_jwt` (the client ID). MUST be the Partner Code you obtained from BMID during on-boarding process (this information is in the onboarding file onboarding file provided by BMID). 
 **aud** | Must be the token endpoint URL
 **jti** | A unique identifier for the token, which can be used to prevent reuse of the token. These tokens MUST only be used once.
 **exp** | Expiration time on or after which the ID Token MUST NOT be accepted for processing.
 
 > Example Request
 
 ```http--inline
 POST /token HTTP/1.1
 Host: server.example.com
 Content-Type: application/x-www-form-urlencoded
 
 grant_type=authorization_code&
 code=SplxlOBeZQQYbYS6WxSbIA&
 redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
 client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer&
 client_assertion=PHNhbWxwOl ... ZT
 ```
 
 ### Successful Token Response
 
 Parameter | Provided | Comment
 -- | -- | --
 **access_token** | Always | A token that can be sent to the UserInfo endpoint to retrieve additional information about the user.
 **token_type** | Always | Will be `Bearer`
 **id_token** | Always | The JWT `id_token` corresponding to the Authentication Request (signed and  encrypted). 
 **at_hash** | Never | Current version of itsme(r) Core does not produce the `at_hash` value
 **refresh_token** | Never | Won't be provided as **itsme(r)** only maintains short-lived session to enforce re-authentication.
 
 ### Errors
 
 As per the OpenID Connect specification [http://openid.net/specs/openid-connect-core-1_0.html#TokenErrorResponse](http://openid.net/specs/openid-connect-core-1_0.html#TokenErrorResponse)
 
 # Requesting User Data
 ## 3. UserInfo Request
 
 As per the OpenID Connect specification [http://openid.net/specs/openid-connect-core-1_0.html#UserInfoRequest](http://openid.net/specs/openid-connect-core-1_0.html#UserInfoRequest)
 
 The UserInfo endpoint can only be accessed with a valid **access_token** received from the Token endpoint during User Authentication, and for a very limited duration after end user authentication; there must be less than 3 minutes between the creation of the user action to be confirmed by the end user on his mobile device, and the access to the UserInfo endpoint.
 
 Your server sends the UserInfo Request using either HTTP `GET` or HTTP `POST`. The Access Token obtained from an Authentication Request must be sent as a Bearer Token. It is recommended that the request use the HTTP `GET` method and the Access Token be sent the using the `Authorization` header field. The HTTP request is sent to the UserInfo endpoint, which you should retrieve from the [Discovery document](https://merchant.itsme.be/oidc/.well-known/openid-configuration) using the key **userinfo_endpoint**.
 
 The Access Token will define the list of Data that will be provided back to the client (TODO). In order to request specific claims, you can [use scopes](#stClaims) in the Authentication Request and/or [use the claims parameter](#Claims-Request) of the **request** Object.
 
 > Example UserInfo Request
 
 ```http--inline
 GET /userinfo HTTP/1.1
 Host: server.example.com
 Authorization: Bearer SlAV32hkKG
 ```
  
 ### Successful UserInfo Response
 
 The content type of the response will be `application/jwt`. The response will be signed and encrypted by BMID using the signing and encryption certificate exposed.
 
 > Example Successful UserInfo Response (Not encrypted nor signed)
 
 ```http--inline
 HTTP/1.1 200 OK
 Content-Type: application/json
 
 {
    "sub": "248289761001",
    "name": "Jane Doe",
    "email": "janedoe@example.com"
 }
 ```
 
 ## <a name="Claims-Scope"></a>Requesting Claims using Scope Values
 
 <aside class="notice"><bold>Note</bold>: itsme(r) won't deliver any <a href="http://openid.net/specs/openid-connect-core-1_0.html#AggregatedDistributedClaims">aggregated nor distributed claims</a> in the current version.</aside>
 
 As per OpenID Connect specification, scopes can be used to request that specific sets of information be made available as Claim Values in UserInfo Token.
 
 In current version and in contradiction to the OpenID Connect specification, **itsme(r)** considers all claims requested via scope as **Essential** (see [Individual Claim Request](http://openid.net/specs/openid-connect-core-1_0.html#IndividualClaimsRequests)). It means the User may not opt out the sharing of specific Data; the User must either gives his consent for the sharing of all Data or refuse the request as a whole. However, as in a future version **itsme(r)**  will make the difference between **Essential** and **Voluntary** claims, you should already request claims with appropriate level regarding your business case. 
 
 The following values for **scope** allow access to predefined sets of Identity Data:

 | Data | Claim
 -- | -- | --
 **`profile`** |given\_name <br> family\_name <br> name <br> gender <br>birthdate <br> locale | `name` and `given_name` and `family_name` 
 | **`email`** | email<br> email_verified<br>| `gender`
 | **`phone`** | phone\_number<br>phone\_number_verified<br>| 
 **`address`** | Address | `address`, with following subfields: `street_address` (newline separator \\n), `locality`, `postal_code`, `country`


 <aside class="notice">
 
 NOTE: Any claim requested by using the scope value can only be obtained from the UserInfo endpoint.</aside>
 
 ## <a name="Claims-Request"></a>Requesting Claims using the "claims" Request Parameter
 We have favoured the request of Data in the **scope** values. However, some specific Data have to be requested in the **claims** parameter of the Authentication Request. Here are these claims:
 
 Data | Claim | Comment
 -- | -- | --
 Subject | **`sub`** | The subject of the `private_key_jwt` (the client ID). Supports value in request.
 Nationality | **`tag:itsmetag:sixdots.be,2016-06:claim_nationality`** | An error will be raised if request as a value element for the claim
 Place of Birth - city | **`tag:itsmetag:sixdots.be,2016-06:claim_city_of_birth`** |  An error will be raised if request as a value element for the claim
 Place of Birth - country | **`tag:itsmetag:sixdots.be,2016-06:claim_country_of_birth`** | An error will be raised if request as a value element for the claim
 E-ID Info  | **`tag:itsmetag:sixdots.be,2016-06:claim_eid`** | Belgian Electronic ID card information encoded in JSON, with the following keys<br>`eid`: the electronic ID card serial number. <br>`issuance_locality`: the issuance locality. <br>`validity_from`: eID card validity “from” date. <br>`validity_to`: eID card validity “to” date. <br>`certificate_validity`: the certificate validity. <br>`read_date`: the data extraction date. <br>Each date is encoded using ISO 8601 UTC (timezone) date format. Example of ISO 8601 UTC date: 2017-04-01T19:43:37+0000
 Passport Number | **`tag:sixdots.be,2017-05:claim_passport_sn`** | Simple string containing the user’s Passport Serial Number.
 Device | **`tag:sixdots.be,2017-05:claim_device`** | see [Device information](#device-information)
 Transaction Info| **`tag:sixdots.be,2017-05:claim_transaction_info`** |Information available in the context of the current transaction.<br> A JSON object with the following keys:<br> (only keys with cardinality \\\[1..1\\\] will be always available)<br> **“securityLevel” \\\[1..1\\\]**: (supported values: <br>{SOFT\\\_ONLY, SIM\\\_ONLY, SIM\\\_AND\\\_SOFT}) Security level used during transaction. <br>**“bindLevel” \\\[1..1\\\]**: (supported values: {SOFT\\\_ONLY, SIM\\\_ONLY, SIM\\\_AND\\\_SOFT}) tells if the user account is bound to a SIM or not, at the time the transaction occurred. <br>**“mcc” \\\[0..1\\\]**: the Mobile Country Code. An Integer (three digits) representing the mobile network country. <br>For example: { "securityLevel": "SIM\\\_AND\\\_SOFT", "bindLevel": "SIM\\\_AND\\\_SOFT", "mcc": 206 }
 E-ID Picture | **`tag:sixdots.be,2017-05:2017-05:claim_photo`**|
 NRN | not supported| 
 
 > Example of Claims request parameter adapted to itsme(r) data
 
 ```json--inline
 {
    "userinfo":
     {
      "given_name": {"essential": true},
      "nickname": null,
      "email": {"essential": true},
      "email_verified": {"essential": true},
      "picture": null,
      "http://example.info/claims/groups": null
     },
    "id_token":
     {
      "auth_time": {"essential": true},
      "acr": {"values": ["urn:mace:incommon:iap:silver"] }
     }
 }
 ```
 
 > Example of JSON device object requested with **tag:sixdots.be,2017-05:claim_device**
 
 ```json--inline
 {  
 	"os": "ANDROID",  
 	"appName": "itsme app", "appRelease": "1.17.13",
 	"deviceLabel": "myDevice",
 	"debugEnabled": false, 
 	"deviceId": "deviceId",
 	"osRelease": "Android 4.4.2", 
 	"manufacturer": "samsung", 
 	"hasSimEnabled": true,
 	"deviceLockLevel": "touchID", 
 	"smsEnabled": true,
 	"rooted": false,
 	"imei": "12345678901234567",
 	"deviceModel": "S8",  
 	"msisdn": "0412123123", 
 	"sdkRelease": "1.17.12"  
 }
 ```
 
 ## Unsupported Standard Claims
 The following standard claims (check http://openid.net/specs/openid-connect-core-1_0.html#StandardClaims) won’t be provided if requested: 
 
  - `middle_name` 
  - `nickname` 
  - `preferred_username` 
  - `picture`
  - `website` 
  - `zoneinfo` 
  - `updated_at`
 
 # Advanced topics
 
 ## <a name="JWTRequest"></a>Passing Request Parameters as JWTs
 
 OpenID Connect defines the following Authentication Request parameters to enable Authentication Requests to be signed and optionally encrypted:
 
 Parameter | Required | Comment
 -- | -- | --
 **request** | Optional | This parameter enables OpenID Connect requests to be passed in a single, self-contained parameter and to be optionally signed and/or encrypted. The parameter value is a Request Object value, as specified in [Section 6.1](http://openid.net/specs/openid-connect-core-1_0.html#RequestObject). It represents the request as a JWT whose Claims are the request parameters.
 **request_uri** | Unsupported | Refer to [JWTRequest](http://openid.net/specs/openid-connect-core-1_0.html#JWTRequests)
 
 The Request Object is a JWT token as defined in [RFC 7519](https://tools.ietf.org/html/rfc7519), which contains at least the following properties:
 
 Property | Required | Comment
 -- | -- | --
 **iss** | Required | Issuer. Must be the `client_id`
 **aud** | Required | Audience. Possible token end-point URLs:<br>https://merchant.itsme.be/oidc/token<br>https://e2emerchant.itsme.be/oidc/token<br>https://uatmerchant.sixdots.be/oidc/token
 
 > Example of claim request before base64url encoding, signing and encryption. In this example, the partners is using the login service. The end user email and nationality will be returned by the UserInfo endpoint.
 
 ```json--inline
 {
 	// JWT Registered claims (https://tools.ietf.org/html/rfc7519#section-4.1)
 	iss: "s6BhdRkqt3",
 	aud: "https://server.itsme.be",
 	// OIDC parameters, must reflect the values of the HTTP parameters
 	client_id: "s6BhdRkqt3",
 	response_type: "code",
 	redirect_uri: "https://client.example.org/cb",
 	scope: "openid email service:PARTNER_LOGIN",
 	state: "af0ifjsldkj",
 	nonce: "n-0S6_WzA2Mj",
 	claims:
 	{
 		userinfo: {
 		    "tag:itsmetag:sixdots.be,2016-06:claim_nationality": null,
 		},
 		id_token: {
 			auth_time: {"essential": true},
 			acr: {"value": \["tag:itsmetag:sixdots.be,2016-06:acr_advanced"\] 		
 		}
 	}
 }
 ```
 
 ## Requests Signing and Encryption 
 
 Encryption algorithm used: RSA SHA-256
 
 Supported algorithms and encryption methods for:
 
 - ID Token
 -- Encryption Method (enc): A128CBC-HS256
 -- Encryption Algorithm (alg): RSA-OAEP
 -- Signing Algorithm (alg): RS256
 - User Info
 -- Encryption Method (enc): A128CBC-HS256
 -- Encryption Algorithm (alg): RSA-OAEP
 -- Signing Algorithm (alg):  RS256
 - Request Object”
 -- Encryption Method (enc): A128CBC-HS256
 -- Encryption Algorithm (alg): RSA-OAEP
 -- Signing Algorithm (alg): RS256
 
 Offline access is not supported. TODO
 
 Dynamic client registration is not allowed.
 
 itsme(r) exposes its signing and encryption keys on a public endpoint (JWKSet) 
 
     https://merchant.itsme.be/oidc/jwkSet
 
 It is expected that the RP will also expose their signing and encryption keys in such a way. The location of the RP JWKSet must be configured by an  administrator of BMID during onboarding of RP. The exposed endpoint must be HTTPS.
 
 
 
 <!--stackedit_data:
 eyJoaXN0b3J5IjpbOTY3NDI5NTk4LC0xMDk5MjI2MTEzLC04OD
 gyMDAxNywtMTMxNzkwMzQzLC04ODgyMDAxNywtMTMxNzkwMzQz
 LC04ODgyMDAxNywtMTMxNzkwMzQzLC04ODgyMDAxNywtMTMxNz
 kwMzQzLDE2NzgxNTEwNzIsLTE0MTEwNzczMDIsMTY3ODE1MTA3
 MiwtMTQxMTA3NzMwMiwxNjc4MTUxMDcyLC0xNDExMDc3MzAyLD
 E2NzgxNTEwNzIsLTE0MTEwNzczMDIsLTkzMzA3MzY3MCwxNjkw
 NzYyMjc2LC05MzMwNzM2NzAsMTY5MDc2MjI3NiwtMTgzMzk2Mz
 kwMCwyMDQ3NTUzODgyLC0xODMzOTYzOTAwLDIwNDc1NTM4ODIs
 LTE4MzM5NjM5MDAsMjA0NzU1Mzg4Ml19+-->
<!--stackedit_data:
eyJoaXN0b3J5IjpbNTAxNDk0MDU3XX0=
-->