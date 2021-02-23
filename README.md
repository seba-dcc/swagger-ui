# Table of Contents

1. [Backgroud](#background)
2. [Actors](#actors)
3. [Architecture](#architecture)
4. [Use Cases](#use-cases)

# Background

## Digital Credentials Consortium

The Digital Credentials Consortium (DCC) is a consortium led by the Massachusetts Institute of Technology (MIT), to which also the Technical University of Munich (TUM) contributes. The goal is "building an infrastructure for digital academic credentials that can support the education systems of the future" [https://digitalcredentials.mit.edu/].

This infrastructure is supposed to increase the efficiency of exchanging and evaluating credentials, provide more reliable ways to protect and verify the credentials, thereby reducing the opportunity for fraud, and expand learner's control over their credentials.

The consortium has released a white paper that describes the features and components of such a system in detail: https://digitalcredentials.mit.edu/wp-content/uploads/2020/02/white-paper-building-digital-credential-infrastructure-future.pdf

In short, as an analogy, a digital credential like a graduation certificate can be imagined as a document within an envelope. The document contains all the relevant information of the credential. The envelope ensures integrity and authenticity of the document. The consortium focuses on the envelope using the concepts of Distributed Identifiers (DID) and Verifiable Credentials (VC).

## DID

> Decentralized identifiers (DIDs) are a new type of identifier that enables verifiable, decentralized digital identity. A DID identifies any subject (e.g., a person, organization, thing, data model, abstract entity, etc.) that the controller of the DID decides that it identifies. In contrast to typical, federated identifiers, DIDs have been designed so that they may be decoupled from centralized registries, identity providers, and certificate authorities. [...] DIDs are URIs that associate a DID subject with a DID document allowing trustable interactions associated with that subject.
> Each DID document can express cryptographic material, verification methods, or service endpoints, which provide a set of mechanisms enabling a DID controller to prove control of the DID.

_W3C Working Draft 20 December 2020: https://www.w3.org/TR/did-core/_

## Verifiable Credential

> Credentials are a part of our daily lives; driver's licenses are used to assert that we are capable of operating a motor vehicle, university degrees can be used to assert our level of education, and government-issued passports enable us to travel between countries. These credentials provide benefits to us when used in the physical world, but their use on the Web continues to be elusive. Currently it is difficult to express education qualifications, healthcare data, financial account details, and other sorts of third-party verified machine-readable personal information on the Web. [...] [The VC] specification provides a standard way to express credentials on the Web in a way that is cryptographically secure, privacy respecting, and machine-verifiable.

_W3C Recommendation 19 November 2019: https://www.w3.org/TR/vc-data-model/_

Typically DIDs are used in VCs as identifiers to unambiguously refer to the object, such as a person, product, or organization by and for which the VCs are issued.

## This Project

The TUM contributes to the DCC by developing an implementation of DIDs using the existing public key infrastructure of TLS certificates for verification of ownership of a DID. For instance, it is verified that a DID for the domain tum.de actually belongs to the TUM. Thus, it is possible to ensure, that a VC issued by this DID was actually issued by the TUM.

Our project is a case within the SEBA Lab Course of the TUM's Software Engineering for Business Information Systems (sebis) chair. The case is about developing a proof of concept of TLS DIDs for the chair. In the end, the proof of concept is supposed to showcase the process of issuing and verifying VCs based on TLS DIDs.

We are Julian Baumann (ge54jaf@mytum.de), Micha Lutz (micha.lutz@tum.de), Riccardo Pagliuca (ge35seq@mytum.de) and Moritz Schindelmann (ge35sub@mytum.de), all Master students at the TUM's Department of Informatics.

# Actors

Following, the three roles in our proof of concept are described.

An issuer is a university that issues a VC to a student. An issuer typically runs a Campus Management System (CAMS) that manages the students, their courses and certificates.

A student (also called holder or learner) is a person that is enrolled at the issuer, received a VC from the issuer and presents this VC to a verifier.

A verifier is an organization like a university or an employer that receives VCs from students and verifies if these VCs are valid. If the verifier is a university, it also typicalls runs a Campus Management System (CAMS).

In our example the issuer is the Technical University of Munich (TUM) with its CAMS called TUMonline. The TUM issues a Bachelor degree to the student, who applies for a Master study at the verifier. The verifier is the Ludwig Maximilian University of Munich (LMU).

# Architecture

The architecture is split into two APIs, corresponding to the two use case scenarios (see below): The issuing part and the verifying part. Both have access to a common Ethereum blockchain.

API documentation: https://seba-dcc.github.io/swagger-ui/

All architecture instances are NodeJS applications. For details please refer to the respective repositories.

## Ethereum Blockchain

The DIDs are stored in the DID registry on an Ethereum blockchain. For development and testing we use a local Ganache blockchain.

The testchain application starts the Ganache blockchain and deploys the relevant contracts from the tls-did-registry repository.

The testchain application uses the ethers, tls-did and tls-did-resolver packages.

Repositories:

- https://github.com/seba-dcc/testchain
- https://github.com/seba-dcc/tls-did-registry

## Issuer API

The Issuer API offers endpoints for creating a DID and for creating a VC. These endpoints are called by the issuer's CAMS. Students have access to the CAMS. The Issuer API has access to the Ethereum blockchain for storing the DIDs.

The Issuer API uses the tls-did and the vc-js packages.

The Issuer API has to run internally on a signing server as it requires access to the issuer's private TLS certificate for creating the issuer's DID.

Repositories:

- https://github.com/seba-dcc/issuer-api

## Verifier API

The Verifier API offers an endpoint for verifying a VC. This endpoint is called by a system (e.g. CAMS) of the verifier. The Verifier API has access to the Ethereum blockchain for resolving the DIDs.

The Verifier API uses the tls-did and the tls-did-resolver packages.

The Verifier API can run anywhere as it does not require access to private information of the verifier. It could run in the verifier's environment, but multiple verifiers could also use a common instance of the Verifier API.

Repositories:

- https://github.com/seba-dcc/verifier-api

# Use Cases

The use cases are split into two scenarios: The issuing part and the verifing part. Each use case corresponds to one endpoint of the Issuer API or Verifier API.

## Issuing Scenario: Create DID

All VCs are related to an issuing DID. Thus, issuers have to create a DID for their domain once in the beginning by calling the Issuer API `/create-did` endpoint from their CAMS.

The endpoint request expects the issuer's domain. Additionally the Issuer API requires access to the private TLS key of this domain for signing the DID, the private Ethereum key for the issuer's blockchain wallet and the address of the DID registry on the blockchain. This information is currently stored in a common config file created when starting the testchain application.

The endpoint stores the DID document in the DID registry on the blockchain and also returns it in the response. The DID document contains information about the id, offered services and offered assertion methods. The id consists of `did:tls:domain`. The offered services contain the issuer's verifiable credential service endpoint. The assertion methods contain the public verification key that is to be used when verifing the validity of a VC issued by this issuer.

The respective public-private key pair is generated while creating the DID. The private key is kept secret and stored locally.

## Issuing Scenario: Create Credential

Issuers can create a verifiable credential for a specific certificate by calling the Issuer API `/create-credential` endpoint from their CAMS.

The endpoint request expects the issuer's domain and a list of requested credentials. A requested credential contains a credential type (e.g. degree) and a credential subject. The credential subject contains the student's DID and the details of the certificate (e.g. the study programme and final grade).

The endpoint returns the list of created verifiable credentials in the response. A verifiable credentials contains a type, the issuer, an issuance date, the credential subject from the request and a proof. The proof contains a JSON Web Signature (JSW) of the content of the verifiable credential based on the locally stored private key that was generated during creation of the DID.

After calling the endpoint, the CAMS would store the signed verifiable credential and offer it as JSON file for download by the student, who can store the file on his computer to later send it to a verifier.

## Verifying Scenario: Verify Credential

Verifiers can verify a verifiable credential that has been sent by a student by calling the Verifier API `/verify-credential` endpoint from their system. E.g. while applying for a study programme the student has uploaded the verifiable credential file to the verifier's CAMS.

The endpoint request expects the verifiable credential file. Additionally it requires access to the address of the DID registry on the blockchain. This information is currently stored in a common config file created when starting the testchain application.

The endpoint returns whether the verifiable credential is valid or not. For this purpose, the issuer's DID document is resolved from the blockchain and following information is validated:

- Integrity of the content of the verifiable credential by resolving its signature with the public key from the issuer's DID document and comparing it against the actual content of the verifiable credential
- Conformity to the context. It is important to know which is the context that the VC uses. A document loader checks all the used contexts and checks for the conformity.
