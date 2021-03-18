---
hip: 0000
title: Using DIDs in Hedera Entity memo fields
author: author@author.com 
type: Standards Track
category: Application
status: Draft
created: 2020-3-16
discussions-to: 
updated: 2020-3-16
requires:
replaces:
superseded-by:
---

## Abstract

This specification provides a standard way to use Decentralized Identifiers (DIDs) within the memo fields of Hedera state entity memo fields. 

## Motivation

Entities (such as accounts, topics, and tokens) on Hedera have a memo field, allowing for an arbitrary string (less than 100 bytes) to be attached to the entity. Entity memos are distinct from memos on transactions, a memo on an entity is persisted in Hedera state.

Decentralized Identifiers (DIDs) [0], as supported by the Hedera DID Method [1], provide a useful format for populating the memo fields. 

According to the W3C, a DID, is “a globally unique identifier that does not require a centralized registration authority because it is registered with distributed ledger technology or other form of decentralized network.”

DIDs can be dereferenced or resolved to a DID Document - a set of data that describes the subject of a DID, including mechanisms, such as public keys that the DID subject can use to authenticate itself and prove their association with the DID. DID Documents are graph-based data structures that are typically expressed using JSON-LD.

Relative to other identifier formats, DIDs have the advantages of  

- Standardized 
- Guaranteed global uniqueness 
- Resolvable into a DID Document carrying useful metadata , e.g. keys , endpoints, etc. 
- Can be cryptographically ‘claimed’ by creating a signature with a private key that corresponds to the public key & signature within the DID Document   
- When their lifecycle is tracked via HCS as in the Hedera DID Method, provable provenance of changes to the DID Document, e.g. for key rotation, introduction of additional endpoints etc. 


## Rationale

When used in a memo field of a Hedera entity, DIDs could be used to 

- Link a Hedera entity to associated metadata, such as an NFT description 
- Link a Hedera entity to an external service, for instance a DID on an HTS token could represent the KYC provider that administers the KYC flag on user accounts 
- Link a Hedera entity to a different public key than of the owner/admin – enabling off-ledger authenticated interactions between that owner and other parties (without reusing Hedera keys) 
- Link two different Hedera entities, for instance an HCS topic and a corresponding HTS token 
- Link a Hedera entity to appropriate certifications about the actors that own or mange that entity, for instance, the renewable energy certifications (using the Verifiable Credentials [2] standard) of an inverter as validated by a 3rd party


## Specification

### Syntax

A DID complying with the Hedera DID method is composed of the following parameters separate by a ':'

 - the string 'did:hedera'
 - a Hedera network identifier, e.g. 'mainnet'
 - the base58-encoded SHA-256 hash of a DID root public key
 - a Hedera file identifier of the application network's address book file listing the member nodes and their IP addresses
 - an optional Hedera topic identifier on which the DID document was registered

Below is an example Hedera DID in which the topic identifier parameter is omitted.

> did:hedera:mainnet:7Prd74ry1Uct87nZqL3ny7aR7Cg46JamVbJgk8azVgUm;hedera:mainnet:fid=0.0.123

### Processes

#### Binding DID to Hedera entity 

1. Create key pair and associated DID as per Hedera DID Method 
2. Register DID into appnet with HCS message carrying DID Document against appropriate HCS topic as per Hedera DID Method 
3. Include DID in memo field when creating or updating entity 

#### Resolving Hedera Entity DID 
 
 1. Query Hedera entity info from mainnet or mirror, e.g. HAPI TokenGetInfo 
 2. Extract DID from memo field 
 3. Resolve DID into DID Document as per Hedera DID Method 
 4. Extract public key from DID Document to verify signatures 
 5. Query listed endpoints for associated data, for instance a Verified Credential  

### Verifiable Credentials

Once a DID has been securely associated with a Hedera entity, then a Verifiable Credential (VC) issued to that DID can be interpreted as having been issued to the entity itself, or the actor that controls the entity. The VC can therefore be seen as having been issued to the entity. 

This secure mapping between entity and VC could be used to

- assert the KYC status of the owner of a Hedera account
- assert the environmental energy generation characteristics of an inverter, and its corresponding REC tokens 
- assert the ?????

When a verifier is presented with a VC, it should be possible to discover the Hedera entity to which the VC was issued. 

The issuer of the VC should include the Hedera entity identifier as a HederaEntityID claim within the VC.

An example VC with a HederaEntityID claim identifying the Hedera entity to which is was issued.

```{
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
  ], 
  "id": "http://example.edu/credentials/1872",
  "type": ["VerifiableCredential", 
           “HederaEntityCredential”],
  "issuer": “did”:hedera:mainnet:dfjkghkdjfgjdjkfgjdnfjgdfg,
  "issuanceDate": "2021-01-01T19:73:24Z", 
  "credentialSubject": {
    "id": "did:hedera:mainnet:dkfgmkdfghdkjfgh”,
    “HederaEntityID: {
      "id": "did:example:c276e12ec21ebfeb1f712ebc6f1",
      "name": [{
        "value": “0.0.”123456,
        "lang": "en"
      }
    }
  },
  
  "proof": {
    
    "type": "RsaSignature2018",  
    "created": "2017-06-18T21:19:10Z",  
    "proofPurpose": "assertionMethod",  
    "verificationMethod": "https://example.edu/issuers/keys/1",  
    "jws": "eyJhbGciOiJSUzI1NiIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..TCYt5X
      sITJX1CxPCT8yAV-TVkIEq_PbChOMqsLfRoPsnsgw5WEuts01mq-pQy7UJiN5mgRxD-WUc
      X16dUEMGlv50aqzpqh4Qktb3rk-BuQy72IFLOqV0G_zS245-kronKb78cPN25DGlcTwLtj
      PAYuNzVBAh4vGHSrQyHUdBBPM"
  }
}
```

## Backwards Compatibility

This HIP is entirely opt-in, and does not break any existing functionality.

## Security Implications

Inserting a DID into an entity's memo field establishes an association between the entity and the controller of the DID, and so a connection between the entity and uses of that DID. For instance, if the same DID is used in a Verifiable Credential, then the identity attributes within that credential may be attributed to the controller of the entity. 

The controller of the DID lays claim to the DID, and so the association to the Hedera entity, with a digital signature using the private key associated with the DID. 

We consider different attacks against the association

### An attacker modifies the memo field to a different DID

The attacker is able to change the value of the DID within an entity's memo field to a DID the attacker controls, thereby claiming the association.

When a Hedera entity is created or updated, the keys that are authorized to osubsequently make changes to the entity can be stipulated. if no such keys are stipulated, then the entity is immutable and the attack is thwarted. If a key is stipulated, then only the admin/owner of the entity can modify the entity and change the memo field. 

Hedera supports multi-signature authorization rules on entity modifications - an entity can be created such that multiple keys must sign a modification like changing the memo field and thereby offering greater resistance to this attack.

### An attacker switches the DID Document that the DID resolves to

The attacker is able to change the DID Document to which a DID in the memo field of an entity resolves into a DID Document that they control, and thereby effectively claim the association. 

The attack is thwarted because only the private key associated with the DID is able to update the DID Document - the update message (as per the Hedera DID method) must have a signature created with the private key associated with the DID's public key.

> signature - A Base64-encoded signature that is a result of signing a minified JSON string of a message attribute with a private key corresponding to the public key #did-root-key in the DID document.

Additionally, in the Hedera DID method, the DID itself is derived from the public key. A verifier would be able to determine that the public key of the attacker did not correspond to the claimed DID.

### An attacker reuses a DID in their entity

The attacker creates a Hedera entity and includes some other actor's DID in the memo field in an attempt to falsely associate the new entity with that other DID.

Nothing prevents the attacker from reusing a DID in this manner - the memo fields on entities are not guaranteed to be unique. 

The attacker will however not have the private key associated with the DID and so will not be able to demonstrate control of the DID. 

Note that it is not sufficient to have the private key sign the DID Document itself at the time of creation.

## How to Teach This

Note that the DID must fit within the memo size restrictions.

## Reference Implementation


## Rejected Ideas

This proposal uses the "memo" field to specify the DID for entities. An alternative would be to use a dedicated DID param added to each entity.

## Open Issues

N/A

## References

[0] https://www.w3.org/TR/did-core/
[1] https://github.com/hashgraph/did-method
[2] https://www.w3.org/TR/vc-data-model/


## Copyright/license

This document is licensed under the Apache License, Version 2.0 -- see [LICENSE](../LICENSE) or (https://www.apache.org/licenses/LICENSE-2.0)
