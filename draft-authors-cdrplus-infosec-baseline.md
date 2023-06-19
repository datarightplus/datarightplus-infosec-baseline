%%%
Title = "CDR+ Security Profile: Baseline"
area = "Internet"
workgroup = "cdrplus-parity"

[seriesInfo]
name = "Internet-Draft"
value = "draft-authors-cdrplus-infosec-baseline-latest"
stream = "IETF"
status = "experimental"

date = 2023-06-16T00:00:00Z

[[author]]
initials="S."
surname="Low"
fullname="Stuart Low"
organization="Biza.io"
  [author.address]
  email = "stuart@biza.io"

[[author]]
initials="B."
surname="Kolera"
fullname="Ben Kolera"
organization="Biza.io"
  [author.address]
  email = "bkolera@biza.io"

%%%

.# Abstract

The CDR+ Security Profile: Baseline is intended to be a compatible profile of the [@!CDS] presented as a profile of [@!FAPI-1.0-Advanced]. This profile focuses primarily on the obligations between OP and RP with respect to authorisation requests and does so as an overlay on the underlying FAPI profile combined with the inclusion of permitted arrangement types within the CDR (currently one, the CDR Sharing Arrangement V1). It does not attempt to provide elaboration on registration protocols, certificate profiles, federation or other components specified within the _[Data Standards: Security Profile](https://consumerdatastandardsaustralia.github.io/standards/#security-profile)_.

.# Notational Conventions

The keywords "**MUST**", "**MUST NOT**", "**REQUIRED**", "**SHALL**", "**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**",  "**MAY**", and "**OPTIONAL**" in this document are to be interpreted as described in [@!RFC2119].

{mainmatter}

# Scope

This document specifies methods for the following:
  - method of obtaining OAuth2 tokens in th designated manner under the CDR Rules and [@!CDS];
  - requirements related to participants for initiating authorisations and obtaining ongoing authorisation credentials

This document does not seek to:
  - specify correlation elements between technical and the legal obligations contained within documents such as the CDR Rules;
  - restate unchanged requirements from upstream specifications
  - incorporate components which are headed towards retirement, notably at the time of drafting, the inclusion of Hybrid authorisation flow

# Terminology

This specification uses the term "JSON Web Token (JWT)" as defined by [@!JWT].

The specification also defines the following terms:

CDR Rules
: Refers to the [Competition and Consumer (Consumer Data Right) Rules 2020](https://www.legislation.gov.au/Series/F2020L00094)

Consumer
: A Consumer is a business or individual who authorises the sharing of data stored by a Data Holder on their behalf to a Software Product with their permission. Beyond having access an individual to their own data a User may also have access to zero or more non-individual Consumer's, for instance businesses that they have permission to make sharing decisions for.

Data Holder
: A Data Holder is a party that holds consumer data for which sharing is to be conducted and for which their customer, the Consumer, participates in the authorisation process initiated by a Data Recipient. Please refer to the expanded description of Data Holder within this document.

Data Recipient
: A Data Recipient is a party that receives consumer data from a Data Holder. This occurs by way of a Software Product.

Ecosystem Authority
: The Ecosystem Authority represents the designated arbiter of trust between Data Holders, Data Recipients and the Consumer. Within the Australian Consumer Data Right this is the Australian Competition and Consumer Commission. Futher elaboration on the Ecosystem Authority is provided within [@!CDRPLUS-ADMISSION-CONTROL].

Personally Identifiable Information (PII)
: Information that (a) can be used to identify the natural person to whom such information relates, or (b) is or might be directly or indirectly linked to a natural person to whom such information relates.

Pairwise Pseudonymous Identifier (PPID)
: Identifier that identifies the Consumer to a Data Recipient that cannot be correlated with the Consumer's PPID at another Data Recipient.

Software Product
: A Software Product represents the technology infrastructure, ostensibly a client registration with an authorisation server, operated by a Data Recipient for the purposes of receiving consumer data.

User
: A User is a human who provides an identifier unique to the individual and for which correlates to one or more Consumer relationships.

User Identifier
: A User Identifier is a unique piece of information, typically a username, which identifies the User

# Data Holder

For the purposes of this specification a Data Holder is described as the party who is offering or has offered services to the Consumer and/or holds relevant data related to those services on behalf of the Consumer.

The types and format of that data is outside the scope of this particular specification but traditionally includes:
- specific customer information such as name, addresses and phone numbers;
- information related to services such as account numbers, pricing information and balances;
- transaction/ledger information pertaining to services provided

Within the CDR ecosystem the mandated Data Holders are ostensible Banking and Energy providers. Additional sectors can be designated by way of a legally binding Designation Instrument coupled with changes to the CDR Rules.

A Software Product assumes the role of an [@!OIDC-Core] OpenID Provider.


## Authorisation Server

The authorisation server **MUST** support the provisions specified in clause 5.2.2 of [@!FAPI-1.0-Advanced] with the following sections replaced as follows:
1. Section 5.2.2-1: **SHALL** accept only PAR issued request object passed by reference using the `request_uri` parameter
1. Section 5.2.2-2: **SHALL** require the `response_type` value `code` in conjunction with the `response_mode` value `jwt`;
1. Section 5.2.2-11: **MUST** support the pushed authorization request endpoint as described in PAR;
1. Section 5.2.2-14: **SHALL** authenticate the confidential client using `private_key_jwt` specified in section 9 of [@!OIDC-Core]

In addition, the authorisation server:

1. **MUST** support the [@!OIDC-Core] scopes `openid` and `profile`;
2. **SHALL** support signed ID Tokens and **MAY** support signed and encrypted ID Tokens;
1. **SHALL** issue access tokens with an expiry time of between 2 and 10 minutes;
1. **MUST NOT** support public clients
1. **SHALL NOT** accept authorisation requests by value
1. **SHALL** provide the lifetime of an access token as an attribute named `expires_in` within the token endpoint response;
1. **MUST** generate the `sub` as a PPID as described in Section 8 of [@!OIDC-Core];
1. **SHALL** issue `request_uri` values which are one-time use and expire between 10 seconds and 90 seconds (this overrides [@RFC9126] clause 4);
1. **MUST NOT** specify duplicate `kid` parameters within the endpoint advertised at `jwks_uri`;
1. **MUST** support a Token End Point as specified in Section 3.1.3 of [@!OIDC-Core];
1. **MUST** support a UserInfo Endpoint as specified in Section 5.3 of [@!OIDC-Core];
1. **MUST** utilise a different Issuer as specified in Section 1.2 of [@!OIDC-Core] for each marketable brand of the Data Holder;
1. **SHALL** support discovery, as defined in OpenID Connect Discovery 1.0 [@!OIDC-Discovery];
1. **SHALL** support an introspection endpoint, as defined in [@!RFC7662];
1. **SHALL** support a revocation endpoint, as defined in [@!RFC7009]
1. **SHALL** not use refresh token rotation unless, in the case a response with a new refresh token is not received and stored by the client, retrying the request (with the previous refresh token) will succeed

### Authorisation Flow

During the authorisation flow, the authorisation server:

1. **MUST** request a User Identifier that can identify the relationship between the user and the Consumer;
1. **MUST** request a User Identifier already known by the User;
1. **MUST** use a one-time password (OTP) provided to the Consumer through an existing channel or mechanism;
1. **MUST NOT** request the User to provide a known password;
1. **MUST NOT** introduce friction designed to make the authorisation process more difficult than it needs to be

#### One-Time Password (OTP)

The One-Time Password (OTP):

1. **MUST** be delivered using existing and preferred channels
1. **MUST** be numeric digits and be between 4 and 6 digits in length
1. **MUST** only be valid for the purposes of establishing authorisations between Data Holder and Software Products
1. **MUST** be invalidated after a reasonable period of time
1. **SHOULD** incorporate a level of pseudo-randomness appropriate for the use case
1. **SHOULD** incorporate controls, such as retry limits, to minimise the risk of enumeration attacks during the authorisation process

### Endpoints

#### Discovery Document

The discovery document from the authorisation server:

1. **MUST** support the `require_pushed_authorization_requests` parameter as described in [@!RFC9126] at the OpenID Connect Discovery 1.0 [@!OIDC-Discovery] endpoint;
1. **SHOULD** support the `require_pushed_authorization_requests` parameter as described in [@!RFC9126] at the [@!RFC8414] endpoint;

#### Introspection Endpoint

The introspection endpoint:

1. **MUST NOT** support introspection of Access Tokens
2. **MUST** include the `sub`, `exp` and `scope` attributes
1. **MUST NOT** include the `username` attribute

#### Revocation Endpoint

The revocation endpoint:
1. **MUST** support revocation of Refresh Tokens
1. **MUST** support revocation of Access Tokens

### Claims

The authorisation server:

1. **MUST** support the [@!OIDC-Core] claims of `sub`, `acr`, `auth_time`, `name`, `given_name` `family_name` and `last_updated`;
2. **MAY** support the [@!OIDC-Core] claims of `email`, `email_verified`, `phone_number` and `phone_number_verified`;
3. **MUST NOT** support any other claims outlined in [@!OIDC-Core];

### Authentication Context Class References (ACR)

The authorisation server **SHOULD** support the following ACR values:

1. `urn:cds.au:cdr:2` where the authentication achieved matches the Credential Level `CL1` rules of [@!TDIF];
2. `urn:cds.au:cdr:3` where the authentication achieved matches the Credential Level `CL2` rules of [@!TDIF]

Additionally, the authorisation server **MUST** ensure all operations meet the requirements of `urn:cds.au:cdr:2` ACR value.

## Resource Server

The resource server provided by Data Holders:
1. **SHALL** support the provisions specified in clause 5.2.2 of [@!FAPI-1.0-Baseline];

# Software Product

For the purposes of this specification, a Software Product is a piece of technology infrastructure managed by a Data Recipient for a specific value proposition. That valuate proposition, following the request of consent from the Consumer, initiates authorisation requests to Data Holders and processes these responses for the purposes of obtaining access credentials (tokens) to utilise at various resource server endpoints.

Within a legal context there are a variety of Data Recipient engagement approaches however this specification seeks to avoid a variety of ambiguity and instead focus on the Software Product itself.

A Software Product assumes the role of an [@!OIDC-Core] Relying Party (Client).

## Authorisation Client

The authorisation client:
1. **SHALL** support the provisions specified in clause 5.2.3 and 5.2.4 of [@!FAPI-1.0-Baseline];
1. **SHALL** support pushed authorisation requests as described in [@!RFC9126];
1. **SHALL** obtain the consent of the Consumer prior to commencing authorisation with the Data Holder;
1. **MUST** submit authorisation requests containing a `exp` claim, in accordance with [@!JWT] Section 4.1.4;
1. **MUST** submit authorisation requests containing a `exp` claim that is less than or equal to `3600`;
1. **MUST** submit authorisation requests containing a `nbf` claim in accordance with [@!JWT] Section 4.1.5

_Note:_ The form and structure of the consent obtained by the authorisation client is outside the scope of this document.

# Supported Arrangement Types

Data Holders and Software Products **MUST** support the following:

1. CDR Sharing Arrangement V1 as described in [@!CDRPLUS-INFOSEC-SHARING-V1]

# TLS Considerations

Section 8.5 of [@!FAPI-1.0-Advanced] **SHALL** apply.

In addition:
1. Use of Mutual TLS is **REQUIRED** at all Authenticated Resource Server endpoints;
1. Use of Mutual TLS is **REQUIRED** at all OAuth2 endpoints except where required for Discovery or Consumer browser access (ie. Authorisation endpoint);
1. All parties **SHALL** utilise certificates issued by the Ecosystem Authority

# Implementation Considerations

Claims available via the profile scope will only return the details of the User which may be different to the Consumer.

Data Holders SHOULD explicitly capture Claims requested by the Data Recipient. If the data cluster or [OIDC] profile scope changes meaning in future this ensures the Data Holder only returns what the Consumer initially authorised to disclose.

Software Products SHOULD record the following information each time an authorisation flow is executed: username (consumer’s ID at the Data Recipient Software Product), timestamp, IP, consent scopes and duration.

# Security Considerations

Data Holders SHOULD implement additional controls to minimise the risk of interception of the OTP through the selected delivery mechanism.

# Acknowledgement

The following people contributed to this document:

- Stuart Low (Biza.io) - Editor
- Ben Kolera (Biza.io)

This document relies heavily upon the [@!CDS] and we therefore acknowledge the contribution of the following individuals:
- James Bligh (Data Standards Body) - Lead Architect for the Consumer Data Right
- Mark Verstege (Data Standards Body) - Lead Architect, Banking & Information Security for the Consumer Data Right
- Ivan Hosgood (Data Standards Body & ACCC) - Solutions Architect

{backmatter}

<reference anchor="OIDC-Core" target="http://openid.net/specs/openid-connect-core-1_0.html"> <front> <title>OpenID Connect Core 1.0 incorporating errata set 1</title> <author initials="N." surname="Sakimura" fullname="Nat Sakimura"> <organization>NRI</organization> </author> <author initials="J." surname="Bradley" fullname="John Bradley"> <organization>Ping Identity</organization> </author> <author initials="M." surname="Jones" fullname="Mike Jones"> <organization>Microsoft</organization> </author> <author initials="B." surname="de Medeiros" fullname="Breno de Medeiros"> <organization>Google</organization> </author> <author initials="C." surname="Mortimore" fullname="Chuck Mortimore"> <organization>Salesforce</organization> </author> <date day="8" month="Nov" year="2014"/> </front> </reference>

<reference anchor="OIDC-Discovery" target="https://openid.net/specs/openid-connect-discovery-1_0.html"> <front> <title>OpenID Connect Discovery 1.0 incorporating errata set 1</title> <author initials="N." surname="Sakimura" fullname="Nat Sakimura"> <organization>NRI</organization> </author> <author initials="J." surname="Bradley" fullname="John Bradley"> <organization>Ping Identity</organization> </author> <author initials="M." surname="Jones" fullname="Mike Jones"> <organization>Microsoft</organization> </author> <author initials="E." surname="Jay"> <organization>Illumila</organization> </author><date day="8" month="Nov" year="2014"/> </front> </reference>

<reference anchor="RFC2119" target="https://datatracker.ietf.org/doc/html/rfc2119"> <front> <title>Key words for use in RFCs to Indicate Requirement Levels</title> <author fullname="S. Bradner"> <organization>Harvard University</organization> </author> </front> </reference>

<reference anchor="RFC7009" target="https://datatracker.ietf.org/doc/html/rfc7009"> <front> <title>OAuth 2.0 Token Revocation</title> <author fullname="T. Lodderstedt, Ed."> <organization>Deutsche Telekom AG</organization> </author><author fullname="M. Scurtescu"> <organization>Google</organization> </author> </front> </reference>

<reference anchor="JWT" target="https://datatracker.ietf.org/doc/html/rfc7519"> <front> <title>JSON Web Token (JWT)</title> <author fullname="M. Jones"> <organization>Microsoft</organization> </author> <author initials="J." surname="Bradley" fullname="John Bradley"> <organization>Ping Identity</organization> </author><author fullname="N. Sakimura"> <organization>Nomura Research Institute</organization> </author> <date month="May" year="2015"/></front> </reference>

<reference anchor="RFC7662" target="https://datatracker.ietf.org/doc/html/rfc7662"> <front> <title>OAuth 2.0 Token Introspection
</title> <author fullname="J. Richer, Ed."> </author> <date month="Oct" year="2015"/></front> </reference>

<reference anchor="RFC8705" target="https://datatracker.ietf.org/doc/html/rfc8705"> <front> <title>OAuth 2.0 Mutual-TLS Client Authentication and Certificate-Bound Access Tokens</title><author fullname="B. Campbell"> <organization>Ping Identity</organization> </author><author initials="J." surname="Bradley" fullname="John Bradley"> <organization>Yubico</organization> </author> <author fullname="N. Sakimura"> <organization>Nomura Research Institute</organization> </author> <author fullname="T. Lodderstedt, Ed."> <organization>YES.com AG</organization> </author><date month="Feb" year="2020"/></front> </reference>

<reference anchor="RFC9126" target="https://datatracker.ietf.org/doc/html/rfc9126"> <front> <title>OAuth 2.0 Pushed Authorization Requests</title> <author fullname="T. Lodderstedt, Ed."> <organization>yes.com</organization> </author><author fullname="B. Campbell"> <organization>Ping Identity</organization> </author><author fullname="N. Sakimura"> <organization>NAT.Consulting</organization> </author> <author fullname="D. Tonge"> <organization>Moneyhub Financial Technology</organization> </author><author fullname="F. Skokan"> <organization>Auth0</organization> </author><date month="Sep" year="2021"/></front> </reference>

<reference anchor="RFC8414" target="https://datatracker.ietf.org/doc/html/rfc8414"> <front> <title>OAuth 2.0 Authorization Server Metadata</title> <author initials="M." surname="Jones" fullname="Mike Jones"> <organization>Microsoft</organization> </author> <author fullname="N. Sakimura"> <organization>NAT.Consulting</organization></author> <author initials="J." surname="Bradley" fullname="John Bradley"> <organization>Yubico</organization> </author><date month="Jun" year="2018"/></front> </reference>

<reference anchor="FAPI-1.0-Advanced" target="https://openid.net/specs/openid-financial-api-part-2-1_0.html"> <front> <title abbrev="FAPI 1.0 Advanced">Financial-grade API Security Profile 1.0 - Part 2: Advanced</title><author initials="N." surname="Sakimura" fullname="Nat Sakimura"><organization>Nat Consulting</organization></author><author initials="J." surname="Bradley" fullname="John Bradley"><organization>Yubico</organization></author> <author initials="E." surname="Jay" fullname="Illumila"><organization>Illumila</organization></author></front> </reference>

<reference anchor="FAPI-1.0-Baseline" target="https://openid.net/specs/openid-financial-api-part-1-1_0.html"> <front><title abbrev="FAPI 1.0 Baseline">Financial-grade API Security Profile 1.0 - Part 1: Baseline</title><author initials="N." surname="Sakimura" fullname="Nat Sakimura"><organization>Nat Consulting</organization></author><author initials="J." surname="Bradley" fullname="John Bradley"><organization>Yubico</organization></author><author initials="E." surname="Jay" fullname="Illumila"><organization>Illumila</organization></author></front> </reference>

<reference anchor="CDRPLUS-INFOSEC-SHARING-V1" target="https://cdrplus.github.io/cdrplus-specs/main/cdrplus-infosec-sharing-v1.html"> <front><title>CDR: Sharing Arrangement V1</title><author initials="S." surname="Low" fullname="Stuart Low"><organization>Biza.io</organization></author></front> </reference>

<reference anchor="CDRPLUS-ADMISSION-CONTROL" target="https://cdrplus.github.io/cdrplus-specs/main/cdrplus-admission-control.html"> <front><title>CDR: Admission Control</title><author initials="S." surname="Low" fullname="Stuart Low"><organization>Biza.io</organization></author></front> </reference>

<reference anchor="CDS" target="https://consumerdatastandardsaustralia.github.io/standards"><front><title>Consumer Data Standards (CDS)</title><author><organization>Data Standards Body (Treasury)</organization></author></front> </reference>

<reference anchor="TDIF" target="https://www.digitalidentity.gov.au"><front><title>Trusted Digital Identity Framework (TDIF)</title><author><organization>Commonwealth of
Australia (Digital Transformation Agency)</organization></author></front> </reference>
