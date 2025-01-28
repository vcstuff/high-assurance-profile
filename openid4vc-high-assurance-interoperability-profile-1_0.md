%%%
title = "OpenID4VC High Assurance Interoperability Profile - draft 01"
abbrev = "openid4vc-high-assurance-interoperability-profile"
ipr = "none"
workgroup = "Digital Credentials Protocols"
keyword = ["security", "openid4vc", "sd-jwt", "sd-jwt-vc", "mdoc"]

[seriesInfo]
name = "Internet-Draft"
value = "openid4vc-high-assurance-interoperability-profile-1_0-02"
status = "standard"

[[author]]
initials="K."
surname="Yasuda"
fullname="Kristina Yasuda"
organization="SPRIND"
   [author.address]
   email = "kristina.yasuda@sprind.org"

[[author]]
initials="T."
surname="Lodderstedt"
fullname="Torsten Lodderstedt"
organization="SPRIND"
   [author.address]
   email = "torsten@lodderstedt.net"

%%%

.# Abstract

This document defines a profile of OpenID for Verifiable Credentials in combination with the credential formats IETF SD-JWT VC [@!I-D.ietf-oauth-sd-jwt-vc] and ISO mdoc [@!ISO.18013-5]. The aim is to select features and to define a set of requirements for the existing specifications to enable interoperability among Issuers, Wallets and Verifiers of Credentials where a high level of security and privacy is required. The profiled specifications include OpenID for Verifiable Credential Issuance [@!OIDF.OID4VCI], OpenID for Verifiable Presentations [@!OIDF.OID4VP], Self-Issued OpenID Provider v2 [@!OIDF.SIOPv2], IETF SD-JWT VC [@!I-D.ietf-oauth-sd-jwt-vc], and ISO mdoc [@!ISO.18013-5].

{mainmatter}

# Introduction

This document defines a set of requirements for the existing specifications to enable interoperability among Issuers, Wallets and Verifiers of Credentials where a high level of security and privacy is required. This document is an interoperability profile that can be used by implementations in various contexts, be it a certain industry or a certain regulatory environment.

This document is not a specification, but a profile. It refers to the specifications required for implementations to interoperate among each other and for the optionalities mentioned in the referenced specifications, defines the set of features to be mandatory to implement.

The profile uses OpenID for Verifiable Credential Issuance [@!OIDF.OID4VCI] and OpenID for Verifiable Presentations [@!OIDF.OID4VP] as the base protocols for issuance and presentation of Credentials, respectively. The credential formats used are IETF SD-JWT VC as specified in [@!I-D.ietf-oauth-sd-jwt-vc] and ISO mdoc [@!ISO.18013-5]. Additionally, considerations are given on how the issuance of Credentials in both IETF SD-JWT VC [@!I-D.ietf-oauth-sd-jwt-vc] and ISO mdoc [@ISO.18013-5] formats can be performed in the same transaction.

A full list of the open standards used in this profile can be found in Overview of the Open Standards Requirements (reference).

## Audience Target audience/Usage

The audience of the document is implementers that require a high level of security and privacy for their solutions. A non-exhaustive list of the interested parties includes [eIDAS 2.0](https://eur-lex.europa.eu/legal-content/EN/TXT/HTML/?uri=OJ:L_202401183), [California Department of Motor Vehicles](https://www.dmv.ca.gov/portal/), [Open Wallet Foundation (OWF)](https://openwallet.foundation/), [IDunion](https://idunion.org/?lang=en), [GAIN](https://gainforum.org/), and [the Trusted Web project of the Japanese government](https://trustedweb.go.jp/en), but is expected to grow to include other jurisdictions and private sector companies.

# Terminology

This specification uses the terms "Holder", "Issuer", "Verifier", "Wallet", and "Verifiable Credential" as defined in @!OIDF.OID4VCI] and [@!OIDF.OID4VP].

# Scope

The following aspects are in scope of this interoperability profile:

* Profile of OpenID4VCI to issue IETF SD-JWT VCs, including
  * Wallet Attestation
* Profile of OpenID4VP to present IETF SD-JWT VCs
* Profile of OpenID4VP over the W3C Digital Credentials API [@w3c.digital_credentials_api] to present
  * IETF SD-JWT VCs
  * ISO mdocs
* Protocol for User Authentication by the Wallet at a Verifier (SIOP v2)
* Profile of IETF SD-JWT VC that includes the following aspects
  * Status management of the Credentials, including revocation
  * Cryptographic Key Binding
  * Issuer key resolution
  * Issuer identification (as prerequisite for trust management)
* Crypto Suites

Note that when OpenID4VP is used, the Wallet and the Verifier can either be remote or in-person.

Assumptions made are the following:

* The issuers and verifiers cannot pre-discover Wallet’s capability
* The issuer is talking to the Wallet supporting the features defined in this profile (via Wallet invocation mechanism)
* There are mechanisms in place for the verifiers and issuers to discover each other’s capability

## Out of Scope

The following items are out of scope for the current version of this document, but might be added in future versions:

* Trust Management, i.e. authorization of an issuer to issue certain types of credentials, authorization of the Wallet to be issued certain types of credentials, authorization of  the Verifier to receive certain types of credentials.
* Protocol for presentation of Verifiable Credentials for offline use-cases, e.g. over BLE.
* Profile of OpenID4VCI to issue ISO mdoc [@!ISO.18013-5] is defined in ISO 23220-3.
* Profile of OpenID4VP without using W3C Digital Credentials API to present ISO mdocs is
defined in [@ISO.18013-7]. For more details, also see Annex B.3 in [@!OIDF.OID4VP].

## Scenarios/Business Requirements

* Combined Issuance of SD-JWT VC and mdoc
* Both issuer-initiated and wallet-initiated issuance
* eIDAS PID and (Q)EAA as defined in eIDAS ARF 1.0

## Standards Requirements

This specification enables interoperable implementations of the following flows:

* Issuance of IETF SD-JWT VC using OpenID4VCI
* Presentation of IETF SD-JWT VC using OpenID4VP
* Presentation of IETF SD-JWT VC using OpenID4VP over W3C Digital Credentials API
* Presentation of ISO mdocs using OpenID4VP over W3C Digital Credentials API

Implementations of this specification do not have to implement all of the flows listed above, but they MUST be compliant to all of the requirements for a particular flow they chose to implement.

# OpenID for Verifiable Credential Issuance

Both the Wallet and the Credential Issuer:

* MUST support the authorization code flow.
* MUST support protocol extensions for the SD-JWT VC credential format profile as defined in (#vc_sd_jwt_profile).
* MUST support sender-constrained tokens using the mechanism defined in [@!RFC9449].
* MUST support [@!RFC7636] with `S256` as the code challenge method.

Both Wallet initiated and Issuer initiated issuance is supported.

## Credential Offer

* The Grant Type `authorization_code` MUST be supported as defined in Section 4.1.1 in [@!OIDF.OID4VCI]
* For Grant Type `authorization_code`, the Issuer MUST include a scope value in order to allow the Wallet to identify the desired Credential Type. The Wallet MUST use that value in the `scope` Authorization parameter.
* As a way to invoke the Wallet, at least a custom URL scheme `haip://` MUST be supported. Implementations MAY support other ways to invoke the Wallets as agreed by trust frameworks/ecosystems/jurisdictions, not limited to using other custom URL schemes.

Note: The Authorization Code flow does not require a Credential Offer from the Issuer to the Wallet. However, it is included in the feature set to allow for Issuer initiated Credential issuance.

Both sending Credential Offer same-device and cross-device is supported.

## Authorization Endpoint

* MUST use Pushed Authorization Requests (PAR) [@!RFC9126] to send the Authorization Request.
* Wallets MUST authenticate itself at the PAR endpoint using the same rules as defined in (#token-endpoint) for client authentication at the token endpoint.
* MUST use the `scope` parameter to communicate credential type(s) to be issued. The scope value MUST map to a specific Credential type. The scope value may be pre-agreed, obtained from the Credential Offer, or the Credential Issuer Metadata.
* The `client_id` value in the PAR request MUST be a string that the Wallet has used as the `sub` value in the client attestation JWT.

## Token Endpoint {#token-endpoint}

* The Wallets MUST perform client authentication as defined in (#wallet-attestation).
* Refresh tokens are RECOMMENDED to be supported for credential refresh. For details, see Section 13.5 in [@!OIDF.OID4VCI].

Note: It is RECOMMENDED to use ephemeral client attestation JWTs for client authentication in order to prevent linkability across Credential Issuers.

Note: Issuers should be mindful of how long the usage of the refresh token is allowed to refresh a credential, as opposed to starting the issuance flow from the beginning. For example, if the User is trying to refresh a credential more than a year after its original issuance, the usage of the refresh tokens is NOT RECOMMENDED.

### Wallet Attestation {#wallet-attestation}

Wallets MUST use wallet attestations as defined in Annex E of [@!OIDF.OID4VCI].

The public key, and optionally a trust chain, used to validate the signature on the Wallet Attestation MUST be included in the `x5c` JOSE header.

## Credential Endpoint

* The `JWT` proof type MUST be supported.

## Server Metadata

* The Credential Issuer MUST publish a mapping of every Credential Type it supports to a scope value.

# OpenID for Verifiable Presentations profile for IETF SD-JWT VC

Requirements for both the Wallet and the Verifier:

* As a way to invoke the Wallet, at least a custom URL scheme `haip://` MUST be supported. Implementations MAY support other ways to invoke the wallets as agreed by trust frameworks/ecosystems/jurisdictions, not limited to using other custom URL schemes.
* Response type MUST be `vp_token`.
* Response mode MUST be `direct_post.jwt`. The Verifier MUST return `redirect_uri` in response to the HTTP POST request from the Wallet, where the Wallet redirects the User to, as defined in Section 7.2 of [@!OIDF.OID4VP]. Implementation considerations for the response mode `direct_post.jwt` are given in Section 12.4 of [@!OIDF.OID4VP].
* Authorization Request MUST be sent using the `request_uri` parameter as defined in JWT-Secured Authorization Request (JAR) [@!RFC9101].
* The Client Identifier Scheme as introduced in Section 5.10 of [@!OIDF.OID4VP] MUST be either `x509_san_dns` or `verifier_attestation`. The Wallet MUST support both. The Verifier MUST support at least one.
* To obtain the issuer's public key for verification, verifiers MUST support Web-based key resolution, as defined in Section 5 of [@!I-D.ietf-oauth-sd-jwt-vc]. The JOSE header `kid` MUST be used to identify the respective key.
* The DCQL query and response as defined in Section 6 of [@!OIDF.OID4VP] MUST be used.

# OpenID for Verifiable Presentations over W3C Digital Credentials API

The following requirements apply for both, the Wallet and the Verifier, unless specified otherwise:

* MUST support Annex A in [@!OIDF.OID4VP] that defines how to use OpenID4VP over the W3C Digital Credentials API.
  * The Wallet MUST support both signed and unsigned requests as defined in Annex A.3.1 and A.3.2 of [@!OIDF.OID4VP]. The Verifier MAY support signed requests, unsigned requests, or both.
* Wallet Invocation is done via the W3C Digital Credentials API or an equivalent platform API. Any other mechanism, including Custom URL schemes, MUST NOT be used.
* Response Mode MUST be `dc_api.jwt`. The response MUST be encrypted.
* Response encryption MUST be performed as specified in [@!OIDF.OID4VP, section 7.3]. The JWE `alg` (algorithm) header parameter (see [@!RFC7516, section 4.1.1])
  value `ECDH-ES` (as defined in [@!RFC7518, section 4.6]), with key agreement utilizing keys on the `P-256` curve (see [@!RFC7518, section 6.2.1.1]) MUST be supported.
  The JWE `enc` (encryption algorithm) header parameter (see [@!RFC7516, section 4.1.2]) value `A128GCM` (as defined in [@!RFC7518, section 5.3]) MUST be supported.
* The DCQL query and response as defined in Section 6 of [@!OIDF.OID4VP] MUST be used. Presentation Exchange as defined in Sections 5.4 and 5.5 of [@!OIDF.OID4VP] MUST NOT be used.

## ISO mdoc specific requirements for OpenID for Verifiable Presentations over W3C Digital Credentials API

Requirements for both the Wallet and the Verifier:

* The Credential Format Identifier MUST be `mso_mdoc`.
* ISO mdoc Credential Format specific DCQL parameters as defined in Annex B.3.1 of [@!OIDF.OID4VP] MUST be used.
* Verifier MAY request more than one Credential in the same request.
* When multiple ISO mdocs are being returned, each ISO mdoc MUST be returned in a separate `DeviceResponse` (as defined in 8.3.2.1.2.2 of [@!ISO.18013-5]), each matching to a respective DCQL query. Therefore, the resulting `vp_token` contains multiple `DeviceResponse` instances.
* The `SessionTranscript` and `Handover` CBOR structures MUST be generated in accordance with Annex B.3.4.1 of [@!OIDF.OID4VP].

## IETF SD-JWT VC specific requirements for OpenID for Verifiable Presentations over W3C Digital Credentials API

Requirements for both the Wallet and the Verifier:

* The Credential Format identifier MUST be `dc+sd-jwt`.
* IETF SD-JWT VC Credential Format specific DCQL parameters as defined in Section 6.4.1 of [@!OIDF.OID4VP] MUST be used.

# Self-Issued OP v2

To authenticate the user, subject identifier in a Self-Issued ID Token MUST be used as defined in [@!OIDF.SIOPv2].

   * As a way to invoke the Wallet, at least a custom URL scheme `haip://` MUST be supported. Implementations MAY support other ways to invoke the Wallets as agreed by trust frameworks/ecosystems/jurisdictions, not limited to using other custom URL schemes.
   * `subject_syntax_types_supported` value MUST be `urn:ietf:params:oauth:jwk-thumbprint`

# SD-JWT VCs {#sd-jwt-vc}

This profile defines the following additional requirements for IETF SD-JWT VCs as defined in [@!I-D.ietf-oauth-sd-jwt-vc].

* Compact serialization MUST be supported as defined in [@!I-D.ietf-oauth-selective-disclosure-jwt]. JSON serialization MAY be supported.
* The following JWT Claims MUST be supported Content (differentiate issuance & presentation)

| Claim | SD-JWT as issued by the Issuer | Normative Definition |
|:--- |:--- |:--- |
|iss |MUST |[@!RFC7519], Section 4.1.1 |
|iat |MUST |[@!RFC7519], Section 4.1.6 |
| exp | SHOULD (at the discretion of the issuer) | [@!RFC7519], Section 4.1.4 |
|cnf|	MUST|	[@!RFC7800]|
|vct|	MUST| [@!I-D.ietf-oauth-sd-jwt-vc]|
|status|SHOULD (at the discretion of the issuer)| [@!I-D.ietf-oauth-status-list]|

* The Issuer MUST NOT make any of the JWT Claims in the table above to be selectively disclosable, so that they are always present in the SD-JWT-VC presented by the Holder.
* It is at the discretion of the Issuer whether to use `exp` claim and/or a `status` claim to express the validity period of an SD-JWT-VC. The Wallet and the verifier  MUST support both mechanisms.
* The `iss` claim MUST be an HTTPS URL. The `iss` value is used to obtain Issuer’s signing key as defined in (#issuer-key-resolution).
* The `vct` JWT claim as defined in [@!I-D.ietf-oauth-sd-jwt-vc].
* The `cnf` claim [@!RFC7800] MUST conform to the definition given in [@!I-D.ietf-oauth-sd-jwt-vc]. Implementations conforming to this profile MUST include the JSON Web Key [@!RFC7517] in the `jwk` sub claim.

Note: Currently this profile only supports presentation of credentials with cryptographic Holder Binding: the holder's signature is required to proof the credential is presented by the holder it was issued to. This profile might support claim-based and biometrics-based holder binding once OpenID for Verifiable Credentials adds support for other forms of Holder Binding. See https://bitbucket.org/openid/connect/issues/1537/presenting-vc-without-a-vp-using-openid4vp

Note: Re-using the same Credential across Verifiers, or re-using the same JWK value across multiple Credentials gives colluding Verifiers a mechanism to correlate the User. There are currently two known ways to address this with SD-JWT VCs. First is to issue multiple instances of the same credentials with different JWK values, so that if each instance of the credential is used at only one Verifier, it can be reused multiple times. Another is to use each credential only once (ephemeral credentials). It is RECOMMENDED to adopt one of these mechanisms.

Note: If there is a requirement to communicate information about the verification status and identity assurance data of the claims about the subject, the syntax defined by [@!OIDF.ekyc-ida] SHOULD be used. It is up to each jurisdiction and ecosystem, whether to require it to the implementers of this profile.

Note: If there is a requirement to provide the Subject’s identifier assigned and maintained by the Issuer, the `sub` claim MAY be used. There is no requirement for a binding to exist between the `sub` and `cnf` claims. See the Implementation Considerations section in [@!I-D.ietf-oauth-sd-jwt-vc].

Note: In some credential types, it is not desirable to include an expiration date (eg: diploma attestation). Therefore, this profile leaves its inclusion to the Issuer, or the body defining the respective credential type.

## Issuer identification and key resolution to validate an issued Credential {#issuer-key-resolution}

This profile supports two ways to represent and resolve the key required to validate the issuer signature of an SD-JWT VC, the web PKI-based key resolution and the x.509 certificates.

* Web-based key resolution: The key used to validate the Issuer’s signature on the SD-JWT VC MUST be obtained from the SD-JWT VC issuer's metadata as defined in Section 5 of [@!I-D.ietf-oauth-sd-jwt-vc]. The JOSE header `kid` MUST be used to identify the respective key.
* x.509 certificates: the SD-JWT VC contains the issuer's certificate along with a trust chain in the `x5c` JOSE header. In this case, the `iss` value MUST be an URL with a FQDN matching a `dNSName` Subject Alternative Name (SAN) [@!RFC5280] entry in the leaf certificate.

Note: The issuer MAY decide to support both options. In which case, it is at the discretion of the Wallet and the Verifier which key to use for the issuer signature validation.

### Cryptographic Holder Binding between VC and VP

* For Cryptographic Holder Binding, a KB-JWT, as defined in [@!I-D.ietf-oauth-sd-jwt-vc], MUST always be present when presenting an SD-JWT VC.

## OpenID4VC Credential Format Profile {#vc_sd_jwt_profile}

A Credential Format Profile for Credentials complying with IETF SD-JWT VCs [@!I-D.ietf-oauth-sd-jwt-vc] is defined in Annex A.3 of [@!OIDF.OID4VCI] and Annex A.4 of [@!OIDF.OID4VP].

## SD-JWT VC Data Model (SD-JWT VCDM)

SD-JWT VCDM is a credential format that is compliant to IETF SD-JWT VC [@!I-D.ietf-oauth-sd-jwt-vc] that allows the usage of existing data structures represented using [@!W3C.VCDM1.1] or [@!W3C.VCDM2.0], while enabling selective disclosure.

SD-JWT VCDM credentials contain a data structure that can be processed using a JSON-LD processor after applying SD-JWT processing.

When IETF SD-JWT VC is mentioned in this specification, SD-JWT VCDM defined in this section MAY be used.

For backward compatibility with JWT processors, the following registered JWT claims MUST be used, instead of their respective counterpart properties in [@!W3C.VCDM1.1] or [@!W3C.VCDM2.0]:

* To represent the validity period of SD-JWT VCDM (i.e., cryptographic signature), `exp`/`iat` Claims encoded as a UNIX timestamp (NumericDate) MUST be used, and not `expirationDate` and `issuanceDate` properties.
* `iss` Claim MUST represent the identifier of the `issuer` property, i.e., the `issuer` property value if `issuer` is a String, or the `id` property of the `issuer` object if `issuer` is an object. `issuer` property MUST be ignored if present.
* `status` Claim MUST represent `credentialStatus` property. `credentialStatus` property MUST be ignored if present.
* `sub` Claim MUST represent the `id` property of `credentialSubject` property. `credentialSubject` property MUST be ignored if present.

IETF SD-JWT VC is extended with the following claims:

* `vcdm`: OPTIONAL. Contains properties defined in [@!W3C.VCDM1.1] or [@!W3C.VCDM2.0] that are not represented by their counterpart JWT Claims as defined above.
The following outlines a suggested non-normative processing model for SD-JWT VCDM:

1. SD-JWT VC Processing:
- A receiver (holder or verifier) of an SD-JWT VCDM applies the processing rules outlined in SD-JWT VC Section 4 of [@!I-D.ietf-oauth-sd-jwt-vc], including verifying signatures, validity periods, status information etc.
- If the `vct` value is associated with any SD-JWT VC Type Metadata, schema validation of the entire SD-JWT VCDM is performed, including the nested `vcdm` claim.
- Additionally, trust framework rules are applied, such as ensuring the issuer is authorized to issue SD-JWT VCDMs for the specified `vct` value.
2. Business Logic Processing:
- Once the SD-JWT VC is verified and trusted by the SD-JWT VC processor, and if the `vcdm` claim is present, the receiver extracts the VCDM (1.1 or 2.0) object from the `vcdm` claim and uses this for the business logic object. If the `vcdm` claim is not present, the entire SD-JWT VC is considered to represent the business logic object.
- The business logic object is then passed on for further use case-specific processing and validation. The business logic assumes that all security-critical functions (e.g., signature verification, trusted issuer) have already been performed during the previous step. Additional schema validation is applied if provided in the `vcdm` claim, e.g., to support SHACL schemas. Note that while a `vct` claim is required, SD-JWT VC type metadata resolution and related schema validation is optional in certain cases.
The following is a non-normative example of an unsecured payload of an SD-JWT VCDM, that is built using the example of unsecured payload in Section 3.3 of [@!I-D.ietf-oauth-sd-jwt-vc]:

```json
{
  "vct": "https://credentials.example.com/identity_credential",
  "vcdm": {    //W3C VCDM 2.0 compliant claims
    "@context": ["https://www.w3.org/ns/credentials/v2"],
    "type": ["VerifiableCredential", "https://credentials.example.com/identity_credential"],
    "credentialSubject": {
      "given_name": "John",
      "family_name": "Doe",
      "email": "johndoe@example.com",
      "phone_number": "+1-202-555-0101",
      "address": {
        "street_address": "123 Main St",
        "locality": "Anytown",
        "region": "Anystate",
        "country": "US"
      },
      "birthdate": "1940-01-01",
      "is_over_18": true,
      "is_over_21": true,
      "is_over_65": true
    }
  }
}
```

The following is a non-normative example of how an unsecured payload of an SD-JWT VCDM above can be used for the SD-JWT:

```json
{
  "_sd": [
    "vwUhdFvpylx9Sqi2YNBV1dfVK6lCbhXvkH0nThfKFT0"
  ],
  "iss": "https://example.com/issuer",
  "iat": 1683000000,
  "exp": 1883000000,
  "vct": "https://credentials.example.com/identity_credential",
  "_sd_alg": "sha-256",
  "cnf": {
    "jwk": {
      "kty": "EC",
      "crv": "P-256",
      "x": "TCAER19Zvu3OHF4j4W4vfSVoHIP1ILilDls7vCeGemc",
      "y": "ZxjiWWbZMQGHVWKVQ4hbSIirsVfuecCE6t4jT9F2HZQ"
    }
  }
}
```

# Crypto Suites

Issuers, holders and verifiers MUST support P-256 (secp256r1) as a key type with ES256 JWT algorithm for signing and signature validation whenever this profiles requires to do so:

* SD-JWT-VC
* Wallet Instance Attestation
* DPoP
* HB JWT
* Authorization request during presentation

SHA256 MUST be supported by all the entities as the hash algorithm to generate and validate the digests in the SD-JWT VC.

Note: When using this profile with other cryptosuites, it is recommended to be explicit about which entity is required to support which curve for signing and/or signature validation

# Implementations Considerations

## Validity Period of the Signature and the Claim Values

`iat` and `exp` JWT claims express both the validity period of both the signature and the claims about the subject, unless there is a separate claim used to express the validity of the claims.

{backmatter}

<reference anchor="OIDF.OID4VCI" target="https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html">
        <front>
          <title>OpenID for Verifiable Credential Issuance</title>
          <author initials="T." surname="Lodderstedt" fullname="Torsten Lodderstedt">
            <organization>yes.com</organization>
          </author>
          <author initials="K." surname="Yasuda" fullname="Kristina Yasuda">
            <organization>Microsoft</organization>
          </author>
          <author initials="T." surname="Looker" fullname="Tobias Looker">
            <organization>Mattr</organization>
          </author>
          <date day="20" month="June" year="2022"/>
        </front>
</reference>

<reference anchor="OIDF.OID4VP" target="https://openid.net/specs/openid-4-verifiable-presentations-1_0-24.html">
      <front>
        <title>OpenID for Verifiable Presentations - draft 24</title>
        <author initials="O." surname="Terbu" fullname="Oliver Terbu">
         <organization>Mattr</organization>
        </author>
        <author initials="T." surname="Lodderstedt" fullname="Torsten Lodderstedt">
          <organization>SPRIND</organization>
        </author>
        <author initials="K." surname="Yasuda" fullname="Kristina Yasuda">
          <organization>SPRIND</organization>
        </author>
        <author initials="T." surname="Looker" fullname="Tobias Looker">
          <organization>Mattr</organization>
        </author>
       <date day="23" month="January" year="2025"/>
      </front>
</reference>

<reference anchor="OIDF.SIOPv2" target="https://openid.net/specs/openid-connect-self-issued-v2-1_0.html">
  <front>
    <title>Self-Issued OpenID Provider V2</title>
    <author ullname="Kristina Yasuda">
      <organization>Microsoft</organization>
    </author>
    <author fullname="Michael B. Jones">
      <organization>Microsoft</organization>
    </author>
   <author initials="T." surname="Lodderstedt" fullname="Torsten Lodderstedt">
      <organization>yes.com</organization>
    </author>
   <date day="18" month="December" year="2021"/>
  </front>
</reference>

<reference anchor="OIDF.ekyc-ida" target="https://openid.net/specs/openid-connect-4-identity-assurance-1_0-ID4.html">
  <front>
    <title>OpenID Connect for Identity Assurance 1.0</title>
    <author ullname="Torsten Lodderstedt ">
      <organization>yes</organization>
    </author>
    <author fullname="Daniel Fett">
      <organization>yes</organization>
    </author>
 <author fullname="Mark Haine">
      <organization>Considrd.Consulting Ltd</organization>
    </author>
     <author fullname="Alberto Pulido">
      <organization>Santander</organization>
    </author>
     <author fullname="Kai Lehmann">
      <organization>1&amp;1 Mail &amp; Media Development &amp; Technology GmbH</organization>
    </author>
     <author fullname="Kosuke Koiwai">
      <organization>KDDI Corporation</organization>
    </author>
   <date day="19" month="August" year="2022"/>
  </front>
</reference>

<reference anchor="ISO.18013-5" target="https://www.iso.org/standard/69084.html">
        <front>
          <title>ISO/IEC 18013-5:2021 Personal identification — ISO-compliant driving license — Part 5: Mobile driving license (mDL)  application</title>
          <author>
            <organization> ISO/IEC JTC 1/SC 17 Cards and security devices for personal identification</organization>
          </author>
          <date year="2021"/>
        </front>
</reference>

<reference anchor="ISO.18013-7" target="https://www.iso.org/standard/82772.html">
        <front>
          <title>ISO/IEC DTS 18013-7 Personal identification — ISO-compliant driving license — Part 7: Mobile driving license (mDL) add-on functions</title>
          <author>
            <organization> ISO/IEC JTC 1/SC 17 Cards and security devices for personal identification</organization>
          </author>
          <date year="2024"/>
        </front>
</reference>

<reference anchor="ISO.23220-3" target="https://www.iso.org/standard/79125.html">
        <front>
          <title>ISO/IEC DTS 23220-3 Cards and security devices for personal identification — Building blocks for identity management via mobile devices</title>
          <author>
            <organization> ISO/IEC JTC 1/SC 17 Cards and security devices for personal identification</organization>
          </author>
          <date year="2023"/>
        </front>
</reference>

<reference anchor="w3c.digital_credentials_api" target="https://wicg.github.io/digital-credentials/">
        <front>
          <title>Digital Credentials API</title>
		  <author fullname="Marcos Caceres">
            <organization>Apple Inc.</organization>
          </author>
          <author fullname="Sam Goto">
            <organization>Google</organization>
          </author>
          <author fullname="Tim Cappalli">
            <organization>Okta</organization>
          </author>
        </front>
</reference>

<reference anchor="VC-DATA" target="https://www.w3.org/TR/vc-data-model-2.0/">
        <front>
        <title>Verifiable Credentials Data Model v2.0</title>
        <author fullname="Manu Sporny">
            <organization>Digital Bazaar</organization>
        </author>
        <author fullname="Dave Longley">
            <organization>Digital Bazaar</organization>
        </author>
        <author fullname="David Chadwick">
            <organization>Crossword Cybersecurity PLC</organization>
        </author>
        <date day="4" month="May" year="2023"/>
        </front>
</reference>

# Combined Issuance of SD-JWT VC and mdocs

* If combined issuance is required, the Batch Credential Endpoint MUST be supported.

# Acknowledgements {#Acknowledgements}

We would like to thank Paul Bastian, Christian Bormann, Mike Jones, Oliver Terbu, Daniel Fett, and Giuseppe De Marco for their valuable feedback and contributions to this specification.

# Notices

Copyright (c) 2023 The OpenID Foundation.

The OpenID Foundation (OIDF) grants to any Contributor, developer, implementer, or other interested party a non-exclusive, royalty free, worldwide copyright license to reproduce, prepare derivative works from, distribute, perform and display, this Implementers Draft or Final Specification solely for the purposes of (i) developing specifications, and (ii) implementing Implementers Drafts and Final Specifications based on such documents, provided that attribution be made to the OIDF as the source of the material, but that such attribution does not indicate an endorsement by the OIDF.

The technology described in this specification was made available from contributions from various sources, including members of the OpenID Foundation and others. Although the OpenID Foundation has taken steps to help ensure that the technology is available for distribution, it takes no position regarding the validity or scope of any intellectual property or other rights that might be claimed to pertain to the implementation or use of the technology described in this specification or the extent to which any license under such rights might or might not be available; neither does it represent that it has made any independent effort to identify any such rights. The OpenID Foundation and the contributors to this specification make no (and hereby expressly disclaim any) warranties (express, implied, or otherwise), including implied warranties of merchantability, non-infringement, fitness for a particular purpose, or title, related to this specification, and the entire risk as to implementing this specification is assumed by the implementer. The OpenID Intellectual Property Rights policy requires contributors to offer a patent promise not to assert certain patent claims against other contributors and against implementers. The OpenID Foundation invites any interested party to bring to its attention any copyrights, patents, patent applications, or other proprietary rights that MAY cover technology that MAY be required to practice this specification.

# Document History

   [[ To be removed from the final specification ]]

   -02

   * Mandate DCQL instead of presentation exchange
   * Refactor HAIP and add details for mdoc profile over DC API
   * Add specific requirements for response encryption
   * Add SessionTranscript requirements
   * Update OID4VP reference to draft 24

   -01

   * Remove the Wallet Attestation Schema and point to OpenID4VCI instead
   * Rename specification to enable non-SD-JWT credential formats to be included
   * Require encrypted responses
   * Remove reference to `client_id_scheme` parameter that no longer exists in OpenID4VP
   * Refresh tokens are now optional

   -00

   *  initial revision
