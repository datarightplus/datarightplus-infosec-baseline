%%%
Title = "DR+ Security Profile: Baseline"
area = "Internet"
workgroup = "datarightplus-parity"

[seriesInfo]
name = "Internet-Draft"
value = "draft-authors-datarightplus-infosec-baseline-latest"
stream = "IETF"
status = "experimental"

date = 2024-02-17T00:00:00Z

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

The DR+ Security Profile: Baseline is intended to be a compatible profile of the [@!CDS] presented as a profile of [@!FAPI-1.0-Advanced]. This profile focuses primarily on the obligations between Provider and Initiator with respect to authorisation requests and does so as an overlay on the underlying FAPI profile combined with the inclusion of specified authorisation types.

This profile does not attempt to provide elaboration on registration protocols, certificate profiles, federation or other components specified within the [@!CDS]. Further terminology used is deliberately jurisdiction agnostic, please refer to [@!DATARIGHTPLUS-ROSETTA] for specific ecosystem mappings.

.# Notational Conventions

The keywords "**REQUIRED**", "**SHALL**", "**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**",  "**MAY**", and "**OPTIONAL**" in this document are to be interpreted as described in [@!RFC2119].

{mainmatter}

# Scope

This document specifies methods for the following:
  - method of obtaining OAuth2 tokens as originally described within the [@!CDS] and;
  - requirements related to participants for initiating authorisations and obtaining ongoing authorisation credentials

This document does not seek to:
  - specify correlation elements between technical and the legal obligations contained within documents such as the CDR Rules;
  - restate unchanged requirements from upstream specifications
  - consider historical obligations which have expired

# Terminology

This specification uses the term "JSON Web Token (JWT)" as defined by [@!JWT] and the terms "Consumer", "Ecosystem Authority", "Initiator", "Personally Identifiable Information (PII)", "Provider", "User" as defined by [@!DATARIGHTPLUS-ROSETTA].

The specification also defines the following terms:

Access Token:
: TODO

ID Token:
: TODO

Issuer
: TODO

Pairwise Pseudonymous Identifier (PPID)
: Identifier that identifies the Consumer to a Initiator that cannot be correlated with the Consumer's PPID at another Initiator.

Refresh Token
: TODO

User Identifier
: A User Identifier is a unique piece of information, typically a username, which identifies the User


## Authorisation Server

The authorisation server **SHALL** support the provisions specified in clause 5.2.2 of [@!FAPI-1.0-Advanced] with the following sections replaced:

1. Section 5.2.2-1: **SHALL** accept only PAR issued request object passed by reference using the `request_uri` parameter;
1. Section 5.2.2-2: **SHALL** require the `response_type` value `code` in conjunction with the `response_mode` value `jwt`;
1. Section 5.2.2-11: **SHALL** support the pushed authorization request endpoint as described in PAR;
1. Section 5.2.2-14: **SHALL** authenticate the confidential client using `private_key_jwt` specified in section 9 of [@!OIDC-Core]

In addition, the authorisation server:

1. **SHALL** support the [@!OIDC-Core] scopes `openid` and `profile`;
2. **SHALL** support signed ID Tokens and **MAY** support signed and encrypted ID Tokens;
1. **SHALL** issue Access Tokens with an expiry time of between 2 and 10 minutes;
1. **SHALL** provide the lifetime remaining of an Access Token as an attribute named `expires_in` within the token endpoint response;
1. **SHALL** generate the `sub` as a PPID as described in Section 8 of [@!OIDC-Core];
1. **SHALL** issue `request_uri` values which are one-time use and expire between 10 seconds and 90 seconds (this overrides [@RFC9126] clause 4);
1. **SHALL NOT** specify duplicate `kid` parameters within the endpoint advertised at `jwks_uri`;
1. **SHALL** support a Token End Point as specified in Section 3.1.3 of [@!OIDC-Core];
1. **SHALL** support a UserInfo Endpoint as specified in Section 5.3 of [@!OIDC-Core];
1. **SHALL** for each Provider, utilise a different Issuer as specified in Section 1.2 of [@!OIDC-Core];
1. **SHALL** support discovery, as defined in OpenID Connect Discovery 1.0 [@!OIDC-Discovery];
1. **SHALL** support an introspection endpoint, as defined in [@!RFC7662];
1. **SHALL** support a revocation endpoint, as defined in [@!RFC7009];
1. **SHALL** ensure all operations meet at at least the requirements of Credential Level `CL1` rules of [@!TDIF];
1. **SHALL NOT** utilise Refresh Token rotation

### Authorisation Flow

During the authorisation flow, the authorisation server:

1. **SHALL** request a User Identifier that can identify the relationship between the user and the Consumer;
1. **SHALL** request a User Identifier already known by the User;
1. **SHALL** use a one-time password (OTP) provided to the Consumer through an existing channel or mechanism;
1. **SHALL NOT** request the User to provide a known password;
1. **SHALL NOT** introduce friction designed to make the authorisation process more difficult than it needs to be

### Endpoints

#### Discovery Document

The discovery document from the authorisation server:

1. **SHALL** support the `require_pushed_authorization_requests` parameter as described in [@!RFC9126] at the OpenID Connect Discovery 1.0 [@!OIDC-Discovery] endpoint;
1. **SHALL** support the `require_pushed_authorization_requests` parameter as described in [@!RFC9126] at the [@!RFC8414] endpoint;

#### Introspection Endpoint

The introspection endpoint:

1. **SHALL NOT** support introspection of Access Tokens
1. **SHALL** include the `sub`, `exp` and `scope` attributes
1. **SHALL NOT** include the `username` attribute

#### Revocation Endpoint

The revocation endpoint:

1. **SHALL** support revocation of Refresh Tokens
1. **SHALL** support revocation of Access Tokens
1. **MAY** support revocation of ID Tokens

### Claims

The authorisation server:

1. **SHALL** support the [@!OIDC-Core] claims of `sub`, `auth_time`, `name`, `given_name` `family_name` and `last_updated`;
3. **SHALL** ignored and not authorise any other claims outlined in [@!OIDC-Core];
1. **SHOULD** support the [@!OIDC-Core] `acr` claim;
2. **MAY** support the [@!OIDC-Core] claims of `email`, `email_verified`, `phone_number` and `phone_number_verified`;

## Resource Server

The resource server provided by Providers:
1. **SHALL** support the provisions specified in clause 5.2.2 of [@!FAPI-1.0-Baseline];

# Initiator

## Authorisation Client

The Initiators authorisation client:

1. **SHALL** support the provisions specified in clause 5.2.3 and 5.2.4 of [@!FAPI-1.0-Baseline];
1. **SHALL** support pushed authorisation requests as described in [@!RFC9126];
1. **SHALL** obtain the consent of the Consumer prior to commencing authorisation with the Provider;
1. **SHALL** submit authorisation requests containing a `exp` claim, in accordance with [@!JWT] Section 4.1.4;
1. **SHALL** submit authorisation requests containing a `exp` claim that is less than or equal to `3600`;
1. **SHALL** submit authorisation requests containing a `nbf` claim in accordance with [@!JWT] Section 4.1.5

_Note:_ The form and structure of the consent obtained by the authorisation client is outside the scope of this document.

# TLS Considerations

Section 8.5 of [@!FAPI-1.0-Advanced] **SHALL** apply.

In addition:

1. Use of Mutual TLS is **REQUIRED** at all Authenticated Resource Server endpoints except where required for Discovery or Consumer browser access (ie. Authorisation endpoint);
1. All parties **SHALL** utilise certificates issued by the Ecosystem Authority

# Implementation Considerations

Claims available via the profile scope will only return the details of the User which may be different to the Consumer.

Providers SHOULD explicitly capture Claims requested by the Initiator. If the data cluster or [OIDC] profile scope changes meaning in future this ensures the Provider only returns what the Consumer initially authorised to disclose.

Initiators SHOULD record the following information each time an authorisation flow is executed:

- User Identifier;
- timestamp;
- IP;
- scopes;
- duration

# Security Considerations

Providers SHOULD implement additional controls to minimise the risk of interception of the OTP through the selected delivery mechanism.

# Acknowledgement

The following people contributed to this document:

- Stuart Low (Biza.io) - Editor
- Ben Kolera (Biza.io)

This document relies heavily upon the [@!CDS] and we therefore acknowledge the contribution of the following individuals:
- James Bligh (Data Standards Body) - Lead Architect for the Consumer Data Right
- Mark Verstege (Data Standards Body) - Lead Architect, Banking & Information Security for the Consumer Data Right
- Ivan Hosgood (formally Data Standards Body & ACCC) - Solutions Architect

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

<reference anchor="DATARIGHTPLUS-INFOSEC-SHARING-V1" target="https://datarightplus.github.io/datarightplus-specs/main/datarightplus-infosec-sharing-v1.html"> <front><title>CDR: Sharing Arrangement V1</title><author initials="S." surname="Low" fullname="Stuart Low"><organization>Biza.io</organization></author></front> </reference>

<reference anchor="DATARIGHTPLUS-ADMISSION-CONTROL" target="https://datarightplus.github.io/datarightplus-admission-control/draft-authors-datarightplus-admission-control.html"> <front><title>DataRight Plus: Admission Control</title><author initials="S." surname="Low" fullname="Stuart Low"><organization>Biza.io</organization></author></front> </reference>

<reference anchor="DATARIGHTPLUS-ROSETTA" target="https://datarightplus.github.io/datarightplus-rosetta/draft-authors-datarightplus-rosetta.html"> <front><title>DR+ Rosetta Stone</title><author initials="S." surname="Low" fullname="Stuart Low"><organization>Biza.io</organization></author></front> </reference>

<reference anchor="CDS" target="https://consumerdatastandardsaustralia.github.io/standards"><front><title>Consumer Data Standards (CDS)</title><author><organization>Data Standards Body (Treasury)</organization></author></front> </reference>

<reference anchor="TDIF" target="https://www.digitalidentity.gov.au"><front><title>Trusted Digital Identity Framework (TDIF)</title><author><organization>Commonwealth of
Australia (Digital Transformation Agency)</organization></author></front> </reference>
