# Status of This Document
This document is not a W3C Standard nor is it on the W3C Standards Track. This is a draft document and may be updated, replaced or obsoleted by other documents at any time. It is inappropriate to cite this document as other than work in progress.

Comments regarding this document are welcome. Please file issues and PRs directly on Github.

Editors:
- Caspar Roelofs, Founder/Director | Gimly caspar@gimly.io
- Jack Tanner, Blockchain and SSI Developer | Gimly jack@gimly.io
- Markus Sabadello | Danube Tech markus@danubetech.com

TODO: turn into a Respect page and host https://respec.org/docs

# Introduction

VerifiableCondition is a new type of verification method for DID Documents. It can be used to express complex conditions and additional meta data about verification methods. It can be used to combine verification methods together to form conjugated conditions such as logical operations &&, thresholds, weighted thresholds, relationships and a delegation to external verification methods.

This new type has been created from discussions during the [Decentralized Identity Foundation](https://identity.foundation)'s ID working group sessions. The need for this type has arisen from the current creation of the EOSIO DID method by [Gimly](https://gimly.io). The type is designed to cover several other important use cases requiring similar conditional logic.

Prior work:
- [Verifiable Conditions planning doc](https://docs.google.com/document/d/1hxEMQxfNuB6Elmd6V-9bEt0kZqSx-DULycn6CjOpMYs) - several VerifiableCondition types have not been added to this draft, see conversation and if you think they are important please submit an issue or PR to add
- [DID core - multisig and delegated use case](https://docs.google.com/presentation/d/1vrmdOnN1tiE54e8h7HyegkJUGyrBUITVFNsAVedUwTE)

## Goals
Create a new verification method type that can:
- Express conditional logic required to validate verification methods
- Express additional metadata about verification methods

## Use Cases

Support for account and key models of the following protocols:
- EOSIO: [Accounts And Permissions](https://developers.eos.io/welcome/latest/protocol-guides/accounts_and_permissions)
- Hyperledger Fabric: [Endorsement policies](https://hyperledger-fabric.readthedocs.io/en/latest/developapps/endorsementpolicies.html?highlight=endorsement%20policy)
- Ripple and BigchainDB: [Composable cryptographic conditionals](https://github.com/rfcs/crypto-conditions)
- KERI: [KERI Thresholds](https://github.com/decentralized-identity/keripy/blob/1b6d25a0ada87a65c6a978336c3a1a273c2e53a6/src/keri/core/coring.py#L3151)
- Hyperledger Indy: [Indy DID Method](https://hackmd.io/@icZC4epNSnqBbYE0hJYseA/S1eUS2BQw)

# The VerifiableCondition Type

```json
{
    "id": "did:example:123#owner",
    "controller": "did:example:123",
    "type": ["VerifiableCondition", "VerifiableConditionAnd"],
    "verificationMethod": {},
}
```

The type property of a verificationMethod which includes "VerifiableCondition" is a verifiable. A verifiable type MUST also specify at least one or more types which specify how the condition is fulfilled.

The “verificationMethod” property is a singular or array value of other verification methods. These can be any valid type including other VerifiableCondition types. In this way, a recursive structure of infinite complexity can be expressed about the cryptographic material required.

The “verificationMethod” can use a [relative DID URL](https://w3c.github.io/did-core/#relative-did-urls) to link to other “verificationMethod”s to avoid duplication in the same DID Document.

## Example
The following DID shows a verificationMethod #1 which requires proofs to match the condition of AND( OR( #1-1-1, #1-1-2), #1-2) to be fulfilled. Note the different types possible.

```json
{
     "@context": [
         "https://www.w3.org/ns/did/v1",
         "https://example.com/did/verifiable-conditions/v1"
     ],
    "id": "did:example:123",
    "verificationMethod": [
        {
            "id": "did:example:123#1",
            "controller": "did:example:123",
            "type": ["VerifiableCondition", "VerifiableConditionAnd"],
            "verificationMethod": [{
                "id": "did:example:123#1-1",
                "controller": "did:example:123",
                "type": ["VerifiableCondition", "VerifiableConditionOr"],
                "verificationMethod": [{
                    "id": "did:example:123#1-1-1",
                    "controller": "did:example:123",
                    "type": "EcdsaSecp256k1VerificationKey2019",
                    "publicKeyBase58": "5JBxKqYKzzoHrzeqwp6zXk8wZU3Ah94ChWAinSj1fYmyJvJS5rT"
                }, {
                    "id": "did:example:123#1-1-2",
                    "controller": "did:example:123",
                    "type": "Ed25519VerificationKey2018",
                    "publicKeyBase58": "PZ8Tyr4Nx8MHsRAGMpZmZ6TWY63dXWSCzamP7YTHkZc78MJgqWsAy"
                }]
            }, {
                "id": "did:example:123#1-2",
                "controller": "did:example:123",
                "type": "Ed25519VerificationKey2018",
                "publicKeyBase58": "H3C2AVvLMv6gmMNam3uVAjZpfkcJCwDwnZn6z3wXmqPV"
            }]
        }
    ]
}
```

This is an example of a [verifiable presentation](https://www.w3.org/TR/vc-data-model/#presentations) that fulfills the above DID's verificationMethod #1 with two signatures #1-1-1 and #1-2.

```json
{
    "@context": [
        "https://www.w3.org/2018/credentials/v1",
        "https://www.w3.org/2018/credentials/examples/v1"
    ],
    "type": "VerifiablePresentation",
    "verifiableCredential": [{
        "@context": [
            "https://www.w3.org/2018/credentials/v1",
            "https://www.w3.org/2018/credentials/examples/v1"
        ],
        "id": "http://example.edu/credentials/1872",
        "type": ["VerifiableCredential", "AlumniCredential"],
        "issuer": "https://example.edu/issuers/565049",
        "issuanceDate": "2010-01-01T19:73:24Z",
        "credentialSubject": {
            "id": "did:example:ebfeb1f712ebc6f1c276e12ec21",
            "alumniOf": {
                "id": "did:example:c276e12ec21ebfeb1f712ebc6f1",
                "name": [{
                "value": "Example University",
                "lang": "en"
                }, {
                "value": "Exemple d'Université",
                "lang": "fr"
                }]
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
    }],
    "proof":  [{
            "type": "JsonWebSignature2020",
            "created": "2020-02-15T17:13:18Z",
            "verificationMethod": "did:example:123#1",
            "proofPurpose": "assertionMethod",
            "jws": "eyJiNjQiOmZhbHNlLCJjcml0IjpbImI2NCJdLCJhbGciOiJFZERTQSJ9.Y0KqovWCPAeeFhkJxfQ22pbVl43Z7UI-X-1JX32CA9MkFHkmNprcNj9Da4Q4QOl0cY3obF8cdDRdnKr0IwNrAw"
        }, {
            "type": "JsonWebSignature2020",
            "created": "2020-02-15T17:13:18Z",
            "verificationMethod": "did:example:123#1",
            "proofPurpose": "assertionMethod",
            "jws": "53Fm3mYG9ZJWwRDgyTt9aRDfpsZCGWycS8VChbvdF7PBgHo6MjKCB5k8.
                    6mV1meQzLKDraHdMzneojsw8eqRYcf7EvXVPC1X2ZRgGHdBHmUhYM4CVt6v4uV4JZ51PWxnB31bM9VvVjaVWu"
        }
    ]
}
```

# Subtypes of VerifiableCondition

## And
```json
{
    "id": "did:example:123#1",
    "controller": "did:example:123",
    "type": ["VerifiableCondition", "VerifiableConditionAnd"],
    "verificationMethod": []
}
```

Fulfilled if all of the verificationMethods provided are fulfilled.

Note: this subtype can be expressed through a Threshold subtype by setting the “threshold” property to the the number of verificationMethods.

## Or
```json
{
    "id": "did:example:123#1",
    "controller": "did:example:123",
    "type": ["VerifiableCondition", "VerifiableConditionOr"],
    "verificationMethod": []
}
```

Fulfilled if any one of the verificationMethods provided are fulfilled.

Note: this subtype can be expressed through a Threshold subtype by setting the “threshold” property to 1.

## Threshold
```json
{
    "id": "did:example:123#4",
    "controller": "did:example:123",
    "type": ["VerifiableCondition", "VerifiableConditionThreshold"],
    "threshold": 3,
    "verificationMethod": [],
}
```

Fulfilled if the number of verificationMethods that are fulfilled are greater than or equal to the “threshold” property.

Note: this subtype can be expressed through a WeightedThreshold subtype by setting all the “weight” properties to 1.

## WeightedThreshold
```json
{
    "id": "did:example:123#5",
    "controller": "did:example:123",
    "type": ["VerifiableCondition", "VerifiableConditionWeightedThreshold"],
    "threshold": 3,
    "verificationMethod": [{
        "verificationMethod": {},
        "weight": 2
    }, {
        "verificationMethod": {},
        "weight": 2
    }, {
        "verificationMethod": {},
        "weight": 1
    }]
}
```

Fulfilled if the sum of the weights of the verificationMethods that are fulfilled are greater than or equal to the “threshold” property.

## Delegated
```json
{
    "id": "did:example:123#10",
    "controller": "did:example:123",
    "type": ["VerifiableCondition", "VerifiableConditionDelegated"],
    "delegatedIdUrl": ""
}
```

Fulfilled if the verificationMethod found by dereferencing the DID URL “delegatedIdUrl” is fulfilled. The dereferenced DID document MUST contain a verificationMethod found using the DID URL. The dereferenced DID document MUST NOT contain multiple verificationMethods found using the DID URL.

An alternative shorthand to this type is to use a DID URI in place that points to a different DID, this can be interpreted as a VerifiableConditionDelegated type. e.g. equivalent verificationMethods:
```json
"did:example:456#key-2"
```
and
```json
{
    "id": "did:example:123#2",
    "controller": "did:example:123",
    "type": ["VerifiableCondition", "VerifiableConditionDelegated"],
    "delegatedIdUrl": "did:example:456#key-2"
}
```

## Relationship
```json
{
    "id": "did:example:123#10",
    "controller": "did:example:123",
    "type": ["VerifiableCondition", "VerifiableConditionRelationship"],
    "parentIdUrl": [],
    "childIdUrl": [],
    "siblingIdUrl": [],
}
```

Has no fulfillment requirements.

Expresses a relationship between different verificationMethods. One of the properties "parentIdUrl", "childIdUrl" or "siblingIdUrl" MUST be present. Each property can be a DID URL or an array of DID URLs.

ParentIdUrl expresses which verificationMethods are a parent to this one.
ChildIdUrl expresses which verificationMethods are a child to this one.
SiblingIdUrl expresses which verificationMethods are a sibling to this one.

This verification Method type is useful to express information for wallet mangement and for proofs.

For [Hierarchical Deterministic (HD) Wallets](https://www.ledger.com/academy/crypto/what-are-hierarchical-deterministic-hd-wallets) or other types of heirachial wallets such as EOSIO, the relationship between different keys in the wallets is required so that the wallets may correctly update its key hierarchy. For example, a parent key is required to update child keys on an EOSIO blockchain.

Such relationship information is useful for wallet management, but can also be used in proofs. For example, an application may require that any approved is for fulfilled if it contains a child key of "did:example:123#1".
