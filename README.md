# OAuth 2.0 Authorization Server with PKCE (Node.js)

## Overview

This project demonstrates a **from-scratch implementation of OAuth 2.0 Authorization Code Flow with PKCE (S256)** using Node.js.  

It includes:

- **auth-server**: Handles user authorization, generates access tokens, and validates PKCE code verifiers.
- **client-app**: Initiates authorization requests and exchanges authorization codes for tokens.
- **resource-server**: Validates access tokens issued by the auth-server; does not trust the client directly.

---

## OAuth 2.0 Flow Diagram

<img width="621" height="542" alt="OAuth2 0-PKCE-flow-chart drawio" src="https://github.com/user-attachments/assets/eff6e1b3-937c-4510-b554-b3a2e66c4c78" />

**How it works:**

1. **Client App** sends authorization request to **Auth Server** with the `code_challenge` which derives from `code_verifier` generated.  
2. **Auth Server** authenticates user and returns an **authorization code**.  
3. **Client App** sends the code + `code_verifier` to **Auth Server** at the token endpoint.  
4. **Auth Server** validates the PKCE verifier and returns an **access token**.  
5. **Client App** uses the access token to access the **Resource Server**
6. **Resource Server** validates the token before returning protected resources.

---

## RSA Key Generation

The auth-server uses RSA keys to **sign and verify tokens**.  

### 1. Generate an RSA private key (PEM format)
```
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out private.pem
```
- Used to sign tokens.
- Standard PEM format.

### 2. Convert the private key to PKCS#8 (PEM, unencrypted)
```
openssl pkcs8 -topk8 -inform PEM -outform PEM \
  -nocrypt -in private.pem -out private_pkcs8.pem
```
- Does not create a new key — PKCS#8 only changes the container format.
- Both private.pem and private_pkcs8.pem contain the same cryptographic material.
- Ensures compatibility and clarity with libraries expecting PKCS#8.

### 3. Extract the public key (PEM format)
```
openssl rsa -in private.pem -pubout -out public.pem
```
- Used by the resource-server to verify signatures.
- Can be shared publicly without exposing the private key.

### Notes
- Newer OpenSSL versions often output PKCS#8 by default, but explicit conversion ensures clarity.
- PKCS#8 conversion does not generate a new key. Identical keys after conversion are expected and correct.

### PKCE (Proof Key for Code Exchange)
- Used in the client-app + auth-server flow to secure public clients.
- The client generates a code_verifier and derives a code_challenge (S256 hash + base64url).
- The auth-server verifies the code_verifier against the stored code_challenge during token exchange.

### Running the Project
1. Generate RSA keys (see above) and place them in auth-server folder.
2. Start the servers:
```
# In separate terminals

cd auth-server
node index.js

cd client-app
node index.js

cd resource-server
node index.js
```
3. Visit client-app in your browser and follow the OAuth flow.

### Running the Tests
```
# In terminal of project root(e.g: C:\{PROJECT_DIRECTORY_PATH}\oauth2-pkce\oauth-2.0\oauth-node-demo\)

npm test
```

### Security Notes
- Never commit private keys (private.pem, private_pkcs8.pem) to GitHub.
- Store secrets (client secrets, environment variables) in .env files and add them to .gitignore.
- PKCE ensures that public clients (like SPA or native apps) can securely exchange authorization codes without a secret.
