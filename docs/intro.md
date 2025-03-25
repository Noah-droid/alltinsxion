# Xion Wallet API Documentation

This document outlines the available endpoints for managing Xion wallets, including wallet generation, recovery, balance checking, and smart contract interactions.

## Base URL

```http
https://xionwallet-8inr.onrender.com
```
### Swagger URL

```http
https://xionwallet-8inr.onrender.com/docs/
```

## Wallet Management Endpoints

### 1. Generate New Wallet
Generate a new Xion wallet with encrypted private key.

```http
POST /generate-wallet
```

#### Response
```json
{
    "address": "xion1...",
    "encryptedPrivateKey": "encrypted-key-string",
    "iv": "initialization-vector",
    "publicKey": "public-key-hex"
}
```

### 2. Generate Wallet from Private Key
Create a wallet using an existing private key.

```http
POST /generate-wallet-service
```

#### Request Body
```json
{
    "privateKey": "1234567890abcdef..."  // 32-byte hex string
}
```

#### Response
```json
{
    "address": "xion1...",
    "encryptedPrivateKey": "encrypted-key-string",
    "iv": "initialization-vector",
    "publicKey": "public-key-hex"
}
```

### 3. Recover Wallet
Recover a wallet using a mnemonic phrase.

```http
POST /recover-wallet
```

#### Request Body
```json
{
    "mnemonic": "word1 word2 word3 ... word12"
}
```

#### Response
```json
{
    "address": "xion1..."
}
```

### 4. Get Wallet Balance
Check the balance of a Xion wallet address.

```http
GET /get-balance/{address}
```

#### Response
```json
{
    "address": "xion1...",
    "balance": "1000 uxion"
}
```

## Smart Contract Interactions

### 1. Query Contract
Query a smart contract without executing a transaction.

```http
POST /contracts/{contract_address}/query
```

#### Request Body
```json
{
    "msg": {
        // Query message object
    }
}
```

#### Response
```json
{
    "success": true,
    "result": {
        // Contract response data
    }
}
```

### 2. Execute Contract
Execute a transaction on a smart contract.

```http
POST /contracts/{contractAddress}/execute
```

#### Request Body
```json
{
    "senderMnemonic": "word1 word2 word3 ... word12",
    "msg": {
        // Execute message object
    },
    "gasLimit": 200000,           // optional
    "gasPrice": "0.025uxion"      // optional
}
```

#### Response
```json
{
    "success": true,
    "result": {
        // Transaction result
    }
}
```

## Important Notes

1. **Security**
   - Private keys and mnemonics should be transmitted securely
   - All requests should use HTTPS
   - Keep mnemonics and private keys safe - they cannot be recovered if lost

2. **Gas and Fees**
   - Default gas limit: 200,000
   - Default gas price: 0.025 uxion
   - Ensure wallet has sufficient balance for transactions

3. **Token Denomination**
   - Balance is shown in `uxion` (micro Xion)
   - 1 XION = 1,000,000 uxion

4. **Error Handling**
   - All endpoints return appropriate HTTP status codes
   - Error responses include descriptive messages
   - Common status codes:
     - 200: Success
     - 400: Bad Request
     - 500: Server Error

5. **Rate Limiting**
   - API endpoints may have rate limiting applied
   - Implement appropriate retry mechanisms in your client

## Example Usage

### Query Contract Balance
```javascript
const response = await fetch('/contracts/xion1.../query', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json'
    },
    body: JSON.stringify({
        msg: {
            balance: {
                address: "xion1..."
            }
        }
    })
});
```

### Execute Token Transfer
```javascript
const response = await fetch('/contracts/xion1.../execute', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json'
    },
    body: JSON.stringify({
        senderMnemonic: "your mnemonic phrase",
        msg: {
            transfer: {
                recipient: "xion1...",
                amount: "1000000"  // 1 XION
            }
        }
    })
});
```