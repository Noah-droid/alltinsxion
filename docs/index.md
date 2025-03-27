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
Create a wallet using an existing private key. This endpoint is particularly useful when you have a pre-generated private key from your backend system and passes it as a request to the api.

#### Use Cases
- Creating deterministic wallets from backend-generated private keys
- Implementing key rotation strategies
- Managing multiple wallets in an enterprise setting

#### Private Key Generation Examples

**Using Node.js Crypto**
```javascript
const crypto = require('crypto');

function generatePrivateKey() {
    const privateKey = crypto.randomBytes(32).toString('hex');
    return privateKey;
}
```

**Using Python**
```python
import secrets

def generate_private_key():
    return secrets.token_hex(32)
```

#### API Endpoint
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

**Using Laravel/PHP**
```php
namespace App\Services;

use Illuminate\Support\Str;
use Illuminate\Support\Facades\Http;

class WalletService
{
    protected $apiBaseUrl = 'https://xionwallet-8inr.onrender.com';

    public function generatePrivateKey(): string
    {
        return bin2hex(random_bytes(32));
    }

    public function createWallet(): array
    {
        try {
            $privateKey = $this->generatePrivateKey();
            
            $response = Http::post($this->apiBaseUrl . '/generate-wallet-service', [
                'privateKey' => $privateKey
            ]);

            if ($response->successful()) {
                // Store wallet data in database
                $wallet = \App\Models\Wallet::create([
                    'address' => $response['address'],
                    'encrypted_private_key' => encrypt($response['encryptedPrivateKey']),
                    'iv' => encrypt($response['iv']),
                    'public_key' => $response['publicKey']
                ]);

                return [
                    'success' => true,
                    'wallet' => $wallet
                ];
            }

            return [
                'success' => false,
                'message' => 'Failed to create wallet'
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'message' => $e->getMessage()
            ];
        }
    }
}
```


#### Complete Implementation Example Using Javascript

```javascript
// Example using React/Node.js
async function createWalletFromPrivateKey() {
    try {
        // 1. Generate private key (backend)
        const privateKey = crypto.randomBytes(32).toString('hex');
        
        // 2. Call the wallet generation service
        const response = await fetch('https://xionwallet-8inr.onrender.com/generate-wallet-service', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({ privateKey })
        });
        
        // 3. Handle the response
        const walletData = await response.json();
        
        // 4. Save wallet data securely
        localStorage.setItem('walletAddress', walletData.address);
        // Note: Store encrypted private key and IV in a secure manner
        secureStorage.set('encryptedPrivateKey', walletData.encryptedPrivateKey);
        secureStorage.set('iv', walletData.iv);
        
        return walletData;
    } catch (error) {
        console.error('Error creating wallet:', error);
        throw error;
    }
}
```

#### Security Considerations
- Never expose private keys in client-side code
- Use secure methods to transmit private keys
- Implement encryption at rest for stored wallet data
- Consider using Hardware Security Modules (HSM) for private key generation in production
- Implement proper access controls and audit logging


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