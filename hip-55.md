---
hip: 55
title: Using DIDs in memo fields
author: paulk@hedera.com 
type: Standards Track
category: Application
status: Draft
created: 2020-3-16
discussions-to: https://github.com/hashgraph/hedera-improvement-proposal/issues/40
updated: 2020-3-08
requires:
replaces:
superseded-by:
---

## Abstract

This specification provides a standard way to use Decentralized Identifiers (DIDs) within the memo fields of Hedera state entity memo fields. 

## Motivation

Entities on Hedera now have a memo field. Decentralized Identifiers (DIDs) [0], as supported by the Hedera DID Method, provide a useful format for populating the memo fields. 

Relative to other identifier formats, DIDs have the advantages of  

- Standardized 
- Guaranteed global uniqueness 
- Resolvable into a JSON DID Document carrying useful metadata , e.g. keys , endpoints, etc. 
- Can be cryptographically ‘claimed’ by creating a signature with a private key that corresponds to the public key & signature within the DID Document   
- When registered via HCS, provable provenance of changes to the DID Document, e.g. for key rotation, introduction of additional endpoints etc. 


## Rationale

When used in a memo field of a Hedera entity, DIDs could be used to 

- Link a Hedera entity to associated metadata, such as an NFT description 
- Link a Hedera entity to an external service, for instance a DID on an HTS token could represent the KYC provider that administers the KYC flag on user accounts 
- Link a Hedera entity to a different public key than of the owner/admin – enabling off-ledger authenticated interactions between that owner and other parties (without reusing Hedera keys) 
- Link two different Hedera entities, for instance an HCS topic and a corresponding HTS token 
- Link a Hedera entity to appropriate certifications about the actors that own or mange that entity, for instance, the renewable energy certifications (using the Verrifiable Credentiqals [1] standard) of an inverter as validated by a 3rd party


## Specification

### Binding DID to Hedera entity 

1. Create key pair and associated DID as per Hedera DID Method 
2. Register DID into appnet with HCS message carrying DID Document against appropriate HCS topic as per Hedera DID Method 
3. Include DID in memo field when creating or updating entity 

### Resolving Hedera Entity DID 
 
 1. Query Hedera entity info from mainnet or mirror, e.g. HAPI TokenGetInfo 
 2. Extract DID from memo field 
 3. Resolve DID into DID Document as per Hedera DID Method 
 4. Extract public key from DID Document to verify signatures 
 5. Query listed endpoints for associated data, for instance a Verified Credential  


## Backwards Compatibility

This HIP is entirely opt-in, and does not break any existing functionality.

## Security Implications

Discuss immutaability?

## How to Teach This

Note that the DID must fit within the memo size restrictions.

## Reference Implementation


## Rejected Ideas

This proposal uses the "memo" field to specify the DID for entities. An alternative would be to use a dedicated DID param added to each entity.

## Open Issues

N/A

## References

[0] https://www.w3.org/TR/did-core/
[1] https://www.w3.org/TR/vc-data-model/


## Copyright/license

This document is licensed under the Apache License, Version 2.0 -- see [LICENSE](../LICENSE) or (https://www.apache.org/licenses/LICENSE-2.0)
