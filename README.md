# Table of Contents

1. [Backgroud](#background)
2. [Actors](#actors)
3. [Architecture](#architecture)
4. [Use Cases](#use-cases)

# Background

## Digital Credentials Consortium

_some information about the consortium and the white paper here..._

https://digitalcredentials.mit.edu/wp-content/uploads/2020/02/white-paper-building-digital-credential-infrastructure-future.pdf

## DID

_some information about the idea of DIDs here..._

https://www.w3.org/TR/did-core/

## Verifiable Credential

_and also some more information VCs here..._

https://www.w3.org/TR/vc-data-model/

## This Project

_here some inforamtion about the TLSDID project of TUM and the purpose of our project in this context..._

# Actors

An issuer is a university that issues a VC to a student. An issuer typically runs a Campus Management System (CAMS) that manages the students, their courses and certificates.

A student (also called holder) is a person that is enrolled at the issuer, received a VC from the issuer and presents this VC to a verifier.

A verifier is an organization like a university or an employer that receives VCs from students and verifies if these VCs are valid. If the verifier is a university, it also typicalls runs a Campus Management System (CAMS).

In our example the issuer is the Technical University of Munich (TUM) with its CAMS called TUMonline. The TUM issues a Bachelor degree to the student, who applies for a Master study at the verifier. The verifier is the Ludwig Maximilian University of Munich (LMU).

# Architecture

The architecture is split into two APIs, corresponding to the two use case scenarios (see below): The issuing part and the verifying part. Both have access to a common Ethereum blockchain.

API documentation: https://seba-dcc.github.io/swagger-ui/

All architecture instances are NodeJS applications. For details please refer to the respective repositories.

## Ethereum Blockchain

The DIDs are stored in the DID registry within an Ethereum blockchain. For development and testing we use a local Ganache blockchain.

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

The Verifier API can run anywhere as it does not require access to private information of the verifier. It could run in the verifier's environment, but multiple verifiers could also use a commong Verifier API.

Repositories:

- https://github.com/seba-dcc/verifier-api

# Use Cases

The use cases are split into two scenarios: The issuing part and the verifing part. Each use case corresponds to one endpoint of the Issuer API or Verifier API.

## Issuing Scenario: Create DID

All VCs are related to an issuing DID. Thus, issuers have to create a DID for their domain once in the beginning by calling the Issuer API `/create-did` endpoint from their CAMS.

The endpoint request expects the issuer's domain. Additionally it requires access to the TLS private key of this domain for signing the DID, the Ethereum private key for the issuer's blockchain wallet and the address of the DID registry in the blockchain. This information is currently stored in a common config file created when starting the testchain application.

The endpoint stores the DID document and also returns it in the response. The DID document contains information about the id, offered services and offered assertion methods. The id consists of `did:tls:domain`. The offered services contain the issuer's verifiable credential service endpoint. The assertion methods contain the public verification key that is to be used when verifing the validity of a VC issued by this issuer.

The respective public-private key pair is generated while creating the DID. The private key is kept secret and stored locally.

_... details about TLS DID ? ..._

## Issuing Scenario: Create Credential

Issuers can create a verifiable credential for a specific certificate by calling the Issuer API `/create-credential` endpoint from their CAMS.

The endpoint request expects the issuer's domain and a list of requested credentials. A requested credential contains a credential type (e.g. a degree) and a credential subject. The credential subject contains the student's DID and the details of the certificate (e.g. the study programme and final grade).

The endpoint returns the list of created verifiable credentials in the response. A verifiable credentials contains a type, the issuer, an issuance date, the credential subject from the request and a proof. The proof contains a JSON Web Signature (JSW) of the content of the verifiable credential based on the locally stored private key that was generated during creation of the DID.

After calling the endpoint, the CAMS would store the signed verifiable credential and offer it as JSON file for download by the student, who can store the file on his computer to later send it to a verifier.

## Verifying Scenario: Verify Credential

Verifiers can verify a verifiable credential that has been sent by a student by calling the Verifier API `/verify-credential` endpoint from their system. E.g. the student has uploaded the verifiable credential file to verifier's CAMS while applying for a study programme.

The endpoint request expects the verifiable credential file. Additionally it requires access to the address of the DID registry in the blockchain. This information is currently stored in a common config file created when starting the testchain application.

The endpoint returns whether the verifiable credential is valid or not. For this purpose, the issuer's DID document is resolved from the blockchain and following information is validated:

- Authenticity of the issuer by checking the issuer's domain with the TLS certificates _--> is this true? how exactly does it work?_
- Integritiy of the content of the verifiable credential by resolving its signature with the public key from the issuer's DID document and comparing it against the actual content of the verifiable credential
