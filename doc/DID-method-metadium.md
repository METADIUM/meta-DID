Metadium DID Method Specification
=================
25th April 2019

# Table of Contents
1. [DID Method Name](#name)
2. [Method Specific Identifier](#identifier)
    1. [Example](#example1)
3. [DID Document](#document)
    1. [Example](#example2)
4. [JSON-LD Context Definition](#ld)
5. [CRUD Operation Definitions](#crud)
    1. [Create (Register)](#create)
    2. [Read (Resolve)](#read)
    3. [Update](#update)
    4. [Delete](#delete)
6. [Security](#security)
7. [Privacy](#privacy)
8. [References](#references)

Metadium is the next-generation identity system powered by blockchain technology. Metadium Decentralized Identifiers is a distributed identifier designed to provide a way for a community connected to the Metadium Ecosystem to uniquely identify an individual, organization, or digital device. The role of a Metadium DID is to provide a service that supports user-authentication and personal information verification. 

The Metadium DID method specification conforms to the requirements specified in 
the DID specification [**[1]**](https://w3c-ccg.github.io/did-spec/), currently published by the 
W3C Credentials Community Group. For more information about DIDs and DID method specifications, 
please see the DID Primer [**[2]**](https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-fall2017/blob/master/topics-and-advance-readings/did-primer.md)

# DID Method Name <a name="name"></a>

The namestring that shall identify this DID method is: `meta`

A DID that uses this method MUST begin with the following prefix: `did:meta`. Per the DID specification, this string MUST be in lowercase. The remainder of the DID, after the prefix, is specified below.

# Method Specific Identifier <a name="identifier"></a>

The method specific identifier is composed of an optional Metadium network identifier with a `:` separator, followed by a Hex-encoded Metadium Identifier Number (MIN) (without a `0x` prefix).
```
meta-did = "did:meta:" + meta-specific-idstring
meta-specific-idstring = meta-network + ":" + MIN
meta-network = "mainnet" | "testnet"
meta-address  = 40*HEXDIG
```
The MIN is case-insensitive, but it is recommended to use mixed-case checksum for address encoding (see [**[3]**](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md)).

## Example <a name="example1"></a>

Example `meta` DIDs:
```
did:meta:fd7022b4B4cAd5eF33723d2C549c85ad196b3db3
did:meta:mainnet:fd7022b4B4cAd5eF33723d2C549c85ad196b3db3
did:meta:testnet:fd7022b4B4cAd5eF33723d2C549c85ad196b3db3
```
# DID Document <a name="document"></a>

## Example <a name="example2"></a>
```
{
	"@context": "https://w3id.org/did/v1",
	"id": "did:meta:testnet:fd7022b4B4cAd5eF33723d2C549c85ad196b3db3",
	"created": "2019-03-25T12:00:00Z",
	"updated": "2019-04-25T12:00:00Z",
	"publicKey": [{
		"id": "key-1",
		"type": ["ECDSA", "secp256r1", "MetaManagementKey"],
		"publicKeyHash": "13F69b6e9d008f2B75437064E1E58EaF597c9ECE"
	}, {
		"id": "key-2",
		"type": ["ECDSA", "secp256r1", "MetaServiceKey"],
		"publicKeyHash": "FB2133404Da4713963EDc84dE0f62a80cF4356d9"
	}, {
		"id": "key-3",
		"type": ["ECDSA", "secp256r1", "MetaServiceKey"],
		"publicKeyHash": "e835bFaaf93b056D0c19AAc38278dc5163f06F15"
	}],
	"authentication": {
		"type": ["ECDSA", "secp256r1"],
		"publicKey": "key-1"
	},
	"service": [{
		"name": "facebook",
		"type": ["ECDSA", "secp256r1"],
		"publicKey": "key-2"
	}, {
		"name": "google",
		"type": ["ECDSA", "secp256r1"],
		"publicKey": "key-3"
	}]
}
```
We use the ISO 8601 [**[4]**](https://www.iso.org/iso-8601-date-and-time-format.html) basic and extended notations for timestamp.
To make an public key hash from the public key, all we need to do is to apply Keccak-256 to the key and then take the last 20 bytes of the result. No Base58 or any other conversion.

# JSON-LD Context Definition <a name="ld"></a>

The `meta` method defines additional JSON-LD terms for the supported Metadium key types `MANAGEMENT`, and `SERVICE`.

The definition of the `meta` JSON-LD context is:
```
{
	"@context":
	{
		"METAManagementKey": "{TBA}",
		"METAServiceKey": "{TBA}"
	}
}
```
Note: Other type of keys, such as `recovery`, `resolver`, and `provider` of a DID Document may be supported in future versions of this specification.

# CRUD Operation Definitions <a name="crud"></a>

## Create (Register) <a name="create"></a>

In order to create a `meta` DID, a Metadium Identity Manager (MIM) smart contract, and Metadium Service Manager (MSM) smart contract must be deployed on Metadium blockchain.
MIM is compliant with the EIP1484 [**[5]**](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1484.md) standard, and MSM is service key resolver.

`meta` DID creation is done by submitting a transaction to the MIM smart contract invoking the following method:
```
function createIdentity(address recoveryAddress, address[] memory providers, address[] memory resolvers) public returns (uint min);
```
This will generate the corresponding id-string (MIN) and assign control to the caller address. Identities are denominated by MINs, which are unique but otherwise uninformative.

## Read (Resolve) <a name="read"></a>

To construct a valid DID document from an `meta` DID, the following steps are performed:

1. Determine the Metadium network identifier ("mainnet", or "testnet"). If the DID contains no network identifier, then the default is "mainnet".
1. Invoke the `function getIdentity(uint min)` function to MIM for getting `MANAGEMENT` key.
1. For each returned key address, look up the associated key.
1. For each `MANAGEMENT` public key hash:
	1. Add a `publicKey` element of type `ECDSA`, `secp256r1` and `MetaManagementKey` to the DID Document.
    1. Add an `authentication` element of type `ECDSA`, and `secp256r1`, referencing this `publicKey`.
1. Invoke the `function getKeys(uint min)` function to MSM for getting  for `SERVICE` key.
1. For each `SERVICE` public key hash:
	1. Add a `publicKey` element of type `ECDSA`, `secp256r1` and `MetaServiceKey` to the DID Document.
	1. Add an `service` element of type  `ECDSA`, `secp256r1`, referencing this `publicKey`.

Note: Service endpoints and other elements of a DID Document may be supported in future versions of this specification.

## Update <a name="update"></a>

The DID Document may be updated by invoking the relevant MSM smart contract functions as follows:
```
function addKey(address _key) public returns (bool success);
function removeKey(address _key) public returns (bool success);
```
## Delete (Revoke) <a name="delete"></a>

Revoking the DID can be supported by executing a `function destructIdentity(uint min)` operation that is part of the MIM smart contract. This will remove the MIM and MSM's storage and code from the state, effectively marking the DID as revoked.

# Security Considerations <a name="security"></a>

TODO

# Privacy Considerations <a name="privacy"></a>

TODO

# References <a name="references"></a>
----------

 **[1]** https://w3c-ccg.github.io/did-spec/

 **[2]** https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-fall2017/blob/master/topics-and-advance-readings/did-primer.md

 **[3]** https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md

 **[4]** https://www.iso.org/iso-8601-date-and-time-format.html

 **[5]** https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1484.md
 
