# Cere Identity/Custody SDK

## Purpose of this project/repo
The tools, API’s, and packaging facilitate a seamless user onboarding process for any application to connect to any Cere or Substrate-based blockchain networks. There is a pre-configured custodial identity service provided by Cere Network to allow immediate setup and testing against the Cere testnet, and to provide as a template for any other network and identity management to be used instead.
### Key components
* Embedded Wallet - An embeddable custodial wallet that is designed to be integrated into any app or website. It is optimized to work with Cere main net, testnet, and any other substrate network. It needs to work with a custodial identity service.
* Pre-configured Cere Identity Service within the embeddable wallet supports auth/key creation/ recovery/etc to generate and custody the identity
* The identity service implements the Transaction Gateway which is the interface that you can use to implement your own identity service (not Cere hosted solution) to work with the embedded wallet.
* Testing tools/ etc.


### Hierarchy
```bash
Hierarchy
├── Embedded Wallet (client-side)
│   └── Abstract Base Wallet
│        ├── Javascript Implementation (already impl)
│        ├── Android implementation
│        └── iOS implementation        
│   
└── Custodial Service Interface (server-side)
│   └── Cere Identity Service Desc/Inst  
│   └── How to impl/extend your own
├── Tools / Tests

```
### Overview diagram
![image](https://drive.google.com/uc?export=view&id=1m3vd6FR9xu0UAoJ2WMg0YxF1XOxDSyJ7)
### 1. Embedded Custody Wallet

#### Responsibilities
* Send onboarding request to Identity Service
* Store key pair in the local storage on the device/browser
  * Users should not be able to use this key pair. It is encrypted by secret configured inside Custody Service.

#### Abstract Base Wallet
```typescript
interface OneTimeTokenResponse {
  accessToken: string
}

interface SessionTokenResponse {
  accessToken: string
}

interface KeyPairResponse {
  publicKey: string,
  privateKey: string,
}

class abstract AbstractBaseWalletOnboardingSDK {
   public function getOneTimeAccessToken(appId: string, externalUserId: string): OneTimeTokenResponse;

   public function getSessionToken(sessionToken: string): SessionTokenResponse;

   public function getKeyPair(sessionToken: string): KeyPairResponse;
}
```
#### Javascript Implementation
##### Repository
* [https://github.com/Cerebellum-Network/Embedded-Wallet-Implementation-JS](https://github.com/Cerebellum-Network/Embedded-Wallet-Implementation-JS)
#### Configuration/Overriding defaults
* This package is configured to talk to Cere Network Identity Service by default. But you can override it to use your own implementation 
* In order to specify a custom `identityURL` you should provide a parameter to the constructor during the object instantiation. Otherwise CERE custody service will be used.
```javascript
const custodySDK = new CustodySDK();
custodySDK.setIdentityEndpoint(“<URL to the Custody Service>”);
```
* In order to specify custom blockchain URL, use `setBlockchainEndpoint` helper (This switching can be implemented using simple dropdown in UI):
```javascript
custodySDK.setBlockchainEndpoint("<URL to your ownBlockchain here>");
```
#### iOS Implementation
* In Progress
#### Android Implementation
* In Progress

### 2. Identity/Custody Service
The server-side services for hosting and custodying of user identities
#### Responsibilities
* Store mapping between user’s wallet Public Key and application User ID
* Provide a possibility to restore private key in case of lost or using on different devices

#### API Interface
Create access token to onboard new user:
```bash
Request: 
// POST 
Endpoint: /onboarding/access-tokens 
Body: {"appId": String, "userId": String}

Response: 
{"accessToken": <Temporary token to send request>}
```
Create new user using temporary access token:
```bash
Request: 
// POST 
Endpoint: /onboarding/users 
Body: {"appId": String, "userId": String}
Headers: {"Authentication": <Temporary token from previous response>} 

Response: 
{"accessToken": <JWT token to get User's data>}
```
Get user’s wallet using JWT token:
```bash
Request: 
// GET
Endpoint: /onboarding/users
Headers: {"Authentication": <JWT token from previous response>} 

Response:
{"publicKey": <User's public key>, "privateKey": <User's private key>}
```
* Here `privateKey` - is an encrypted key, which can be used for signing transactions only. 
### Pre-Configured Cere Identity Service
* Cere Network provides the service, which implements the API interface described in the [section](#api-interface) above. 
* This is a hosted service designed to work with Cere Test Net. 
* You can override this in order to work with any Substrate compatible Networks. 
* In order to use your own Identity Service implementation, you need to override this endpoint as described [here](#javascript-implementation).


### 3. Tools/Tests
This section consist of some useful Tools
#### Example application with Embedded Wallet
##### Repository
* https://github.com/Cerebellum-Network/Embedded-Wallet-App-Example
##### Quick Start
```bash
git clone https://github.com/Cerebellum-Network/Embedded-Wallet-App-Example
npm start
```
