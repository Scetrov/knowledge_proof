# Chapter 4: Authentication with Sui zkLogin

## 4.1 Token model

Use a two-level model:

1. **zkLogin JWT** (from Sui infra):

   - Proves a web2 login and binds it to a Sui address.
   - Used once at login.

2. **App JWT** (your token):

   - Signed with your own secret/keypair.
   - Contains:

     - `sub`: internal `user.id`
     - `addr`: Sui address
     - Optional: `iat`, `exp`

   - Used for all calls from frontend → Rust API.

## 4.2 Auth flow

1. User visits `/login` on the frontend.
2. Clicks "Sign in with Sui":

   - Frontend redirects to zkLogin provider (Sui) with client id, redirect URI, etc.

3. zkLogin redirects back to `/auth/callback` with tokens / proof.
4. Frontend calls an internal route (e.g. Next.js API route `/api/auth/complete`) with zkLogin result.
5. That route calls Rust `POST /api/auth/exchange`:

   - Payload: zkLogin token/proof.

6. Rust API:

   - Verifies zkLogin token.
   - Extracts Sui address and subject.
   - Finds or creates user in DB (by `sui_address`).
   - Issues an app JWT.
   - Returns app JWT.

7. Next.js:

   - Stores app JWT in an `HttpOnly` cookie (via `Set-Cookie` from the API route), or other secure method.

8. Future requests:

   - Frontend API client attaches `Authorization: Bearer <app_jwt>` to calls.

9. Rust API middleware:

   - Validates app JWT signature.
   - Extracts `user_id`, `sui_address`.
   - Makes user available to handlers via request extensions.

You can ask the LLM to:

> Design the exact request/response schema for `POST /api/auth/exchange`, and sketch Axum handlers plus middleware to validate an app JWT with `jsonwebtoken`.
