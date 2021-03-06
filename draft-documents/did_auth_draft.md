# DID Auth - An Overview

Slack: [https://weboftrustinfo.slack.com/messages/C9N0M37ST](https://weboftrustinfo.slack.com/messages/C9N0M37ST) (#did-auth)


# Authors

Markus Sabadello (markus@danubetech.com)

Kyle Den Hartog (kyle.denhartog@evernym.com)

Christian Lundkvist (christian.lundkvist@consensys.net)

Cedric Franz (cedric.franz@gmail.com)

Alberto Elias (hi@albertoelias.me)

Andrew Hughes (AndrewHughes3000@gmail.com)

John Jordan (john.jordan@gov.bc.ca)


# Resources

Markus' topic paper "DID Auth: Scope, Formats, and Protocols":

[https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-spring2018/blob/master/topics-and-advance-readings/DID%20Auth:%20Scope,%20Formats,%20and%20Protocols.md](https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-spring2018/blob/master/topics-and-advance-readings/DID%20Auth:%20Scope,%20Formats,%20and%20Protocols.md)

Kyle's topic paper "DID-Auth protocol":

[https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-spring2018/blob/master/topics-and-advance-readings/DID-Auth%20protocol.md](https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-spring2018/blob/master/topics-and-advance-readings/DID-Auth%20protocol.md)

Uport specification for Selective Disclosure request/response protocol:

[https://github.com/uport-project/specs/blob/develop/flows/selectivedisclosure.md](https://github.com/uport-project/specs/blob/develop/flows/selectivedisclosure.md)

[https://github.com/uport-project/specs/blob/develop/messages/sharereq.md](https://github.com/uport-project/specs/blob/develop/messages/sharereq.md)

[https://github.com/uport-project/specs/blob/develop/messages/shareresp.md \
](https://github.com/uport-project/specs/blob/develop/messages/shareresp.md)


# Abstract

The term DID Auth has been used in different ways and is currently not well-defined. We define DID Auth as a ceremony where an _identity owner_, with the help of various components such as web browsers, mobile devices, and other agents, cryptographically proves to a _relying party_ that they are in control of a DID. This means demonstrating control over the DID using the mechanism specified in the DID Document's "authentication" object. This could take place using a number of different data formats, protocols, and flows. DID Auth includes the ability to establish mutually authenticated communication channels, and to authenticate to web sites and applications. Authorization, Verifiable Credentials, and Capabilities are built on top of DID Auth and out of scope for this document. This paper gives on overview of the scope of DID Auth, supported protocols and flows, the use of components of the DID Documents which are relevant to authentication, as well as formats for challenges and responses.


# Definitions

**TODO:** Review this a bit more.

**Authentication**: The ceremony where an identity owner proves to a _relying party_ that the _identity owner_ controls a DID, by a mechanism that is described in the DID's associated DID Document.

**Authorization**: Method of verifying the rights and privileges of an individual to perform certain actions upon a DID that is not proving its ownership.

**Decentralized Identifier (DID)**: A globally unique identifier that does not require a centralized registration authority because it is registered with distributed ledger technology or other form of decentralized network. (see [here](https://w3c-ccg.github.io/did-spec/#terminology))

**DID Document**: A set of data that describes a DID, including mechanisms, such as public keys and pseudonymous biometrics, that an entity can use to authenticate itself as the DID. A DID Document may also contain other attributes or claims describing the entity. (see [here](https://w3c-ccg.github.io/did-spec/#terminology))

**DID Record**: The combination of a DID and its associated DID Document.

**Identity owner**: Individual, organization or thing who has rights and privileges to perform certain actions upon a DID.

**Identity subject**: Individual, organization or thing whom a DID identifies.

**Relying party**: Individual, organization or thing that authenticates an Identity Owner using the DID Auth protocol. Also called "Verifier" in other specifications. 

**Verifiable Credentials**: A set of one or more claims which are statements made by an issuer about a subject that is tamper-resistant and whose authorship can be cryptographically verified (see [here](https://w3c.github.io/vc-data-model/#terminology)).


# Introduction

**TODO** Review diagrams.

Diagrams by Markus:

[DID Auth Diagrams](./media/DID_Auth_Diagrams.odg)

![DID Auth Diagrams](./media/DID_Auth_Diagrams.png)

Original drawing by Cedric:

![DID Auth scope](./media/DID_Auth.jpg)

## Scope

DID Auth defines data formats and challenge and response transports allowing an _identity owner_ to prove control over a DID to a _relying party_. A successful DID Auth interaction may create the required conditions to allow the parties to exchange further data in a trustworthy way. This further data could include streams of raw data from sensors to the exchange of Verifiable Credentials. This further exchange of data is out of scope of the DID Auth exchange and specification. 

Proof of control over a DID is a technical action which may be a precursor to establishing a trustworthy relationship between two parties. The DID Auth interaction may be a one way interaction where A proves control over a DID<sub>A</sub> to B, or a two way interaction where mutual proof of control over DIDs is achieved. Two way control is a situation where A proves control over DID<sub>A</sub> to B and B proves control over DID<sub>B</sub> to A.

It is in the purview of the two parties engaged in the interaction to determine the need to have a one way or two way DID Auth interaction. It is also in the purview of the two parties to determine if further exchanges of data may be necessary to establish the nature of the relationship between the two parties. 

Implementers may decide to subsume a DID Auth interaction within a higher layer interaction such as the exchange of a Verifiable Credential which could simultaneously prove control over a DID and offer some other Verifiable Credentials for some specific transaction specific purpose.

## Generic Ceremony

This section describes a conceptual authentication framework elaborating the specific flows of the DID Auth protocol. It relates general concepts of authentication to specific concepts of DID Auth.

**Authentication of a DID means: An identity owner demonstrates knowledge or possession or control of the entity's authentication material that was generated and distributed during DID Record Creation through execution of the authentication proof mechanism.**

There are two main phases in DID Auth: **DID Record Creation**, and **Authentication of a DID**.

## DID Record Creation

As stated in the DID Spec, the steps to create a DID Record are:

1.  Generate a NEW_DID as specified in the relevant DID Method Specification.
1.  Generate a NEW_DID_DOCUMENT as specified in the relevant DID Method Specification.
    1.  The id property must have the value of NEW_DID.
1.  Choose one or more authentication type(s) from the array of proof mechanisms.
    1.  Record the type property in an `authentication` object of the NEW_DID_DOCUMENT.
1.  Generate authentication material for use at a later time during Authentication of a DID. The authentication type determines how to generate authentication material for a proof mechanism.
1.  Communicate and store the authentication material, either directly or as derived material, in the NEW_DID_DOCUMENT and for storage by the Identity Owner. If the chosen proof mechanism is based on asymmetric keys, the authentication material in the NEW_DID_DOCUMENT is recorded in a `publicKey` object in the DID Document.

Example `authentication` and `publicKey` objects in a DID Document:

```
{
	"@context": "https://w3id.org/did/v1",
	"id": "did:example:123456789abcdefghi",
	"authentication": [{
		"type": "RsaSignatureAuthentication2018",
		"publicKey": "did:example:123456789abcdefghi#keys-1"
	}, {
		"type": "Ed25519SignatureAuthentication2018",
		"publicKey": "did:example:123456789abcdefghi#keys-2"
	}],
	"publicKey": [{
		"id": "did:example:123456789abcdefghi#keys-1",
		"type": "RsaVerificationKey2018",
		"owner": "did:example:123456789abcdefghi",
		"publicKeyPem": "-----BEGIN PUBLIC KEY...END PUBLIC KEY-----\r\n"
	}, {
		"id": "did:example:123456789abcdefghi#keys-2",
		"type": "Ed25519VerificationKey2018",
		"owner": "did:example:123456789abcdefghi",
		"publicKeyBase58": "H3C2AVvLMv6gmMNam3uVAjZpfkcJCwDwnZn6z3wXmqPV"
	}]
}
```

DID Auth can use different transports for exchanging challenges and responses between an _identity owner_ and a _relying party_. One such transport uses an HTTP POST call to a DID Auth service endpoint. This service endpoint can be discovered from a DID Document.

Example DID Auth service endpoint in a DID Document:

```
{
	"@context": "https://w3id.org/did/v1",
	"id": "did:example:123456789abcdefghi",
	"service": {
		"type": "DidAuthService",
		"serviceEndpoint": "https://auth.example.com/did:example:123456789abcdefg"
	}
}
```

## Authentication of a DID

Note: This flow describes a _relying party_ authenticating the DID of an _identity owner_. 

1.  Determine which DID requires authentication (THE_DID).
1.  Resolve THE_DID to THE_DID_DOCUMENT using a DID resolver that implements the DID Method Specification relevant to THE_DID.
1.  Confirm that the id property of THE_DID_DOCUMENT equals THE_DID.
1.  Select the authentication method to use for DID Auth from the list contained in THE_DID_DOCUMENT. The authentication method chosen must be supported by the relying party.
1.  Establish a transport mechanism between the identity owner and the relying party over which DID Auth will happen (e.g. DID Auth service endpoint lookup, QR code scan, etc).
1.  Send challenge to identity owner over established transport mechanism. Must include a nonce (to prevent replay attacks), and selected authentication method as a minimum and optionally a callback transport mechanism.
1.  Identity owner signs challenge using the selected method and produces a signed response.
1.  Identity owner sends the signed response back to relying party.
1.  Initiator using the public key of THE_DID_DOCUMENT and verifies the signed response of the identity owner.
1.  Optionally use the other content of the DID Document to perform other steps.

Note: Steps 7,8,9 make the assumption that asymmetric keys are used for the proof mechanism.


# Challenge Formats

In a DID Auth interaction, a challenge is transmitted by a _relying party_ to an _identity owner_, asking the _identity owner_ to return a response that proves their control of a DID. This section documents data formats for such challenges.

 * The _relying party_ may or may not know the _identity owner_'s DID at the time the challenge is constructed, and therefore the _identity owner_'s DID may or may not be included in the challenge.
 * The challenge that is sent by the _relying party_ may or may not itself contain a proof of the _relying party_'s control of a DID.
 * The _relying party_ may or may not need additional transport-specific information about the _identity owner_ in order to be able to deliver the challenge (e.g. a DID Auth service endpoint). This additional protocol-specific information may be discoverable from the _identity owner's_ DID that is known to the relying party.
 * The _identity owner_ may or may not need additional transport-specific information about the _relying party_ in order to be able to deliver the response (e.g. a callback URL). This additional protocol-specific information may be included in the challenge, or it may be discoverable from the _relying party_'s DID that is included in the challenge.

## Example DID Auth challenge (in JWT format)

JWT payload:

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJFUzI1NksifQ.eyJpc3MiOiIyb3BpTnJHU3hUVzZzRm82ZXJ2SFpuRWVFWEdOR0FvdXBGTiIsImlhdCI6MTUyMDQ2ODg4MywidmVyaWZpZWQiOlsiTXkgVGl0bGUiXSwiY2FsbGJhY2siOiJodHRwOi8vMTkyLjE2OC40NC4xNjI6ODA4MS9jYWxsYmFjayIsImV4cCI6MTUyMDQ2OTE4MywidHlwZSI6InNoYXJlUmVxIn0.zZVwQ-KT5XdLPXmXOOe2UgGFpWiq5MA7QVoLnIvcLE5p-KJfQ2Yvvhg5hWbrgRF5RzcpDvV7jyRKzBe-_6aMIQ
```

Header and payload decodes to:

```
{
  "typ": "JWT",
  "alg": "ES256K"
}
{
  "iss": "did:uport:2opiNrGSxTW6sFo6ervHZnEeEXGNGAoupFN",
  "iat": 1520468883,
  "callback": "http://192.168.44.162:8081/callback",
  "exp": 1520469183,
  "type": "shareReq"
}
```

## Example DID Auth challenge (in JSON-LD format)

```
{
	"id": "???",
	"type": ["Credential"],
	"issuer": "???",
	"issued": "2018-03-07",
	"claim": {
		"id": "???",
		"publicKey": ""
	}
}
```


# Response Formats

A DID Auth response is constructed by an _identity owner_ after reception of a DID Auth challenge. This section documents data formats for such responses.

A DID Auth response includes the _identity owner_'s DID together with proof of the identity owner's control of that DID.

In order to validate the response and therefore complete a DID Auth flow, the _relying party_ performs the following steps:

 * The _relying party_ resolves the _identity owner_'s DID to its DID Document and interprets the `authentication` and `publicKey` objects in that DID Document to determine how to verify the proof.
 * The _relying party_ must be able to internally link a response to a prior challenge. This can be done with a nonce in the challenge that must also be included in the response. It can also be done by including the entire original challenge in the response.

## Example DID Auth response (in JWT format)

```
{
  "typ": "JWT",
  "alg": "ES256K"
}
{
  "iss": "did:uport:2omWsSGspY7zhxaG6uHyoGtcYxoGeeohQXz",
  "iat": 1520469645,
  "aud": "did:uport:2opiNrGSxTW6sFo6ervHZnEeEXGNGAoupFN",
  "type": "shareResp",
  "nad": "2omWsSGspY7zhxaG6uHyoGtcYxoGeeohQXz",
  "req":
"eyJ0eXAiOiJKV1QiLCJhbGciOiJFUzI1NksifQ.eyJpc3MiOiIyb3BpTnJHU3hUVzZzRm82ZXJ2SFpuRWVFWEdOR0FvdXBGTiIsImlhdCI6MTUyMDQ2OTYzMCwidmVyaWZpZWQiOlsiTXkgVGl0bGUiXSwiY2FsbGJhY2siOiJodHRwOi8vMTkyLjE2OC4xLjE5OTo4MDgxL2NhbGxiYWNrIiwiZXhwIjoxNTIwNDY5OTMwLCJ0eXBlIjoic2hhcmVSZXEifQ.54K3IHiCkgjigTSnUiVqAQZ575vpSMNcGlaKeCxo8ZeghObvaVuvDaE7ToWDiJK8VtQUAOj3pY40tRomon8zAA", // This is a copy of the Request (this is to support a stateless architecture on the server side)
  "exp": 1520556045
}
```

## Example DID Auth response (in JSON-LD Verifiable Credentials format)

The following Verifiable Credential contains a claim of a certain public key as a property of the _identity owner_.

```
{
	"id": "...",
	"type": ["Credential", "DIDAuthCredential"],
	"issuer": "did:example:123456789abcdefghi",
	"issued": "2018-03-07",
	"claim": {
		"id": "did:example:123456789abcdefghi",
		"publicKey": "did:example:123456789abcdefghi#keys-2"
	},
	"proof": {
		"type": "Ed25519Signature2018",
		"created": "2018-01-01T21:19:10Z",
		"creator": "did:example:123456789abcdefghi#keys-2",
		"nonce": "c0ae1c8e-c7e7-469f-b252-86e6a0e7387e",
		"signatureValue": "..."
	}
}
```


# Challenge Transports

A DID Auth challenge may be delivered by a _relying party_ to an _identity owner_ in different ways. DID Auth defines a few common ways this can be done.

 * Multiple devices, user agents, and other technical components may act on behalf of the _identity owner_ to receive and process the challenge. For example, an identity owner's DID Auth service endpoint may receive the challenge and relay it to the _identity owner_'s mobile app.

## DID Auth service endpoint

If the _identity owner_'s DID is known to the _relying party_, it resolves it to its associated DID Document to find a DID Auth service endpoint and sends an HTTP POST with the challenge to that endpoint.

## Scan QR code from mobile app

The _relying party_ may display a DID Auth challenge encoded as a QR code, which is delivered to the _identity owner_ by scanning it with a DID Auth compatible mobile app.

## Mobile deep link

A mobile app or webpage may itself be a _relying party_ and deliver a DID Auth challenge to another mobile app that represents the _identity owner_.

Example:

```
didauth:?challenge=...
```

## Custom protocol handler

Similar to a mobile deep link, a web page can contain a web-based protocol link that contains a DID Auth challenge.

Example:

```
<a href="web+didauth:challenge=?...">Login with DID Auth</a>
```

## Device-to-device communication

If both the _relying party_ and _identity owner_ meet physically with devices they control, a DID Auth challenge can be delivered via direct communication between devices, using Bluetooth, NFC, WiFi, etc.

## Invoke user agent's JavaScript API

A _relying party_ web site may deliver a DID Auth challenge to the _identity owner_ by invoking an API via a JavaScript function in the _identity owner_'s user agent.

**TODO:** Add example from Browser Credential Handler API

**TODO:** Add example from WebAuthn


# Response Transports

## HTTP POST to callback URL

The DID Auth response may be delivered to the _relying party_ in the form of an HTTP POST to a callback URL. This callback URL may be known to the _identity owner_ in advance, it may have been included in the challenge, or it may be discoverable from the _relying party_'s DID.

## Scan QR code from mobile app

The _identity owner_ may encode a DID Auth response as a QR code, which is delivered to the _relying party_ by scanning it with a DID Auth compatible mobile app.

## Hardware wallet

The DID Auth challenge may be forwarded to a hardware wallet, that upon an _identity owner_'s physical interaction, will create a response and send it back to the _relying party_ via the forwarder.


# Architectures

Based on the above data format and transport options for DID Auth challenges and responses, this section describes example overall architectures that can be realized.

**TODO**: In this section, add flows from Markus' topic paper

[https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-spring2018/blob/master/topics-and-advance-readings/DID%20Auth:%20Scope,%20Formats,%20and%20Protocols.md](https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-spring2018/blob/master/topics-and-advance-readings/DID%20Auth:%20Scope,%20Formats,%20and%20Protocols.md)

**TODO**: In this section, add scenarios from BCGov DID Auth opportunity:

[http://bit.ly/2I86vvR](http://bit.ly/2I86vvR)

1.  **System to System:** Services that generate verifiable claims about organizations and write them to TheOrgBook need to be authenticated. Such Services must have a DID known to TheOrgBook and use that DID to access the Issuer API for writing verifiable claims to TheOrgBook. This would be a generic authentication method between two services at the API level.
1.  **Administrative:** There is an administrative element of TheOrgBook that should be limited to authorized users who would have a DID.
    1.  This use case could (should) be expandable into the use case where an arbitrary user can connect to a service for the purposes of "enrolling" and subsequently re-connecting once they are known to the service 

**TODO**: In this section, add info on DID-TLS and TLS Felx

**TODO**: In this section, add info on HTTP Signatures.

**TODO**: In this section, add Kyle's DID Auth protocol.

[https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-spring2018/blob/master/topics-and-advance-readings/DID-Auth%20protocol.md](https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-spring2018/blob/master/topics-and-advance-readings/DID-Auth%20protocol.md)

**TODO**: In this section, add Identity Hub Attestation Flows and Components

https://hackmd.io/9TuwUcHFTUq6Su_WYUH46Q#

https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-spring2018/blob/master/draft-documents/Identity%20Hub%20Attestation%20Handling.md

## Peer to Server

(also known as "Button on Website")

## Peer to Peer

## Server to Server (DID-TLS)


# Relation to other Technologies

## Biometrics

**TODO**: Explain use of biometrics for unlocking private keys on a local device. Also explain use of a BOPS (or similar) service endpoint.

https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-spring2018/blob/master/draft-documents/Biometrics.md

## WebAuthn

**TODO**

IIW #26 Session Notes "WebAuthn + DID Auth": http://iiw.idcommons.net/WebAuthn_%2B_DID_Auth

Web Authentication: An API for accessing Public Key Credentials Level 1

[https://www.w3.org/TR/webauthn/#authentication](https://www.w3.org/TR/webauthn/#authentication)

## OpenID Connect

**TODO**

IIW #26 Session Notes "Open ID v. FIDO v. SSI": http://iiw.idcommons.net/Open_ID_v._FIDO_v._SSI

IIW #26 Session Notes "DID Auth Workflows (Part 2)": http://iiw.idcommons.net/DID_Auth_Workflows_(Part_2)

## Verifiable Credentials Exchange

**TODO:** Explain possible relationship between DID Auth and Verifiable Credentials Exchange protocols. Maybe include Markus' three ways of thinking about it:

https://lists.w3.org/Archives/Public/public-credentials/2018Mar/0077.html


# Security and Privacy Considerations

## Support for multiple DIDs

During a DID Auth interaction, a _relying party_ may request that the _identity owner_'s user agent display a DID selection dialog where the _identity owner_ is able to pick the DID to authenticate with to the _relying party_. The DID Auth challenge and response are then done using that DID.

An _identity owner_'s user agent may also automatically choose (or even dynamically create) different DIDs for each _relying party_. This idea is also known as directed identity. If a different DID is used for each _relying party_, then they are called pairwise-pseudonymous DIDs. This measure is helpful (but not always sufficient) to avoid correlation of the _identity owner_.

## Automatic authentication

Depending on the challenge and response transports, and depending on the user agents, it may be possible to automate some steps in a DID Auth interaction. For example, a DID Auth browser extension can automatically complete a DID Auth interaction to log in to a website using a single click on a button.

## Re-authentication

Even after a DID Auth interaction has been completed, a _relying party_ may require an _identity owner_ to re-authenticate in certain cases, e.g. when a high-value transaction such as transferring money is initiated.

## DID resolution

DID Auth depends on the ability to resolve a DID to its associated DID Document. Therefore all security considerations associated with DID resolution must be taken into account. Among other topics, this includes caching behavior of a DID resolver, as well as metadata about the DID resolution process (e.g. has the DID Document been retrieved via a blockchain full node, or via an untrusted intermediary lookup service).

## Single log-out

DID Auth should provide ways to end all sessions and channels that have been established for an authenticated DID. An _identity owner_ should be able to trigger single log-out manually using their user agent. Single log-out should also happen automatically if a DID is revoked, which means that _relying parties_ must monitor authenticated DIDs for revocation.


# Open Questions

Do we assume that there is a standard TLS channel with a website or do we want to use the DIDs to set up an authenticated and/or encrypted channel?

Can there be a DID Auth - OAuth proxy service that allows a relying party to implement standard OAuth2, provided by a gateway service that implements a DID Auth relying party?
