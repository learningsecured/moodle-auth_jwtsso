# `auth_jwtsso` Example Identity Provider (IdP)

This directory contains a working example Identity Provider for Moodle’s **JWT SSO authentication plugin** (`auth_jwtsso`).

It demonstrates how a real IdP can issue signed JWTs for Moodle, including the proper **nonce flow** for replay protection.

## ⚙️ Requirements

* **Python 3.8+**
* `pip install flask requests pyjwt cryptography`

From the plugin root:

```bash
cd auth/jwtsso/examples
pip install flask requests pyjwt
```

## 🚀 Running the Example IdP

Start the example IdP locally:

```bash
python3 idp_example.py
```

Expected output:

```
Example IdP running at http://localhost:5000/sso/login
Try visiting: http://localhost:5000/sso/login?email=behat@example.com
```

It runs a small Flask web server that simulates an external Identity Provider.

## 🔐 How the Flow Works

This example implements the *full IdP-initiated flow* supported by `auth_jwtsso`:

| Step | Actor             | Description                                                                                                                                                                                   |
| ---- | ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1️⃣  | **IdP**           | Fetches a valid nonce from Moodle’s `/auth/jwtsso/start.php` endpoint.                                                                                                                        |
| 2️⃣  | **IdP**           | Creates a signed JWT containing: <br>• `iss` (issuer) <br>• `aud` (audience) <br>• `iat` / `exp` <br>• `nonce` (the one from Moodle) <br>• user claims (`email`, `given_name`, `family_name`) |
| 3️⃣  | **IdP**           | Redirects the browser to Moodle’s `/auth/jwtsso/callback.php?token=<jwt>`                                                                                                                     |
| 4️⃣  | **Moodle plugin** | Validates the signature, issuer, audience, time window, and nonce, then logs the user in.                                                                                                     |

## 🧩 Configuration in Moodle

Before testing, configure your plugin settings:

| Setting                        | Example value                                                 |
| ------------------------------ | ------------------------------------------------------------- |
| **Issuer (iss)**               | `https://reimann-dev.ddns.net/test-idp`                       |
| **Audience (aud)**             | `https://reimann-dev.ddns.net/`                               |
| **Allowed algorithms**         | `RS256`                                                       |
| **Public key (PEM)**           | Paste the contents of `auth/jwtsso/tests/fixtures/public.pem` |
| **Just-in-time user creation** | ✅ Enabled                                                     |
| **Show login button**          | ✅ Optional                                                    |

## 🧪 Test the Flow

Once the IdP is running, open in your browser:

```
http://localhost:5000/sso/login?email=behat@example.com
```

Expected result:

* The IdP requests a nonce from Moodle.
* It signs and sends a JWT to `/auth/jwtsso/callback.php`.
* Moodle validates and logs in as **Behat User**.

Terminal log example:

```
→ Requesting fresh nonce from https://reimann-dev.ddns.net/auth/jwtsso/start.php
✓ Got nonce: 4d97b7cfcf2142e28dcdcb6a1b9ef7c1
→ Redirecting to: https://reimann-dev.ddns.net/auth/jwtsso/callback.php?token=eyJhbGciOi...
```

## 🔍 Understanding the JWT

The generated JWT contains the following claims:

```json
{
  "iss": "https://reimann-dev.ddns.net/test-idp",
  "aud": "https://reimann-dev.ddns.net/",
  "iat": 1761990800,
  "exp": 1761991400,
  "nonce": "4d97b7cfcf2142e28dcdcb6a1b9ef7c1",
  "email": "behat@example.com",
  "given_name": "Behat",
  "family_name": "User"
}
```

All values must match the plugin configuration for validation to succeed.

## 📘 Implementation Notes

* The script reads the test private key from:

  ```
  auth/jwtsso/tests/fixtures/private.pem
  ```
* The matching public key should be configured in Moodle’s settings.
* This IdP uses RS256 signing; other algorithms (ES256, etc.) can be supported with minor changes.
* You can easily adapt this Python code to integrate with frameworks like **Odoo**, **FastAPI**, or **Django**.

## ⚠️ Security Disclaimer

This example is for **testing and development only**.
Never deploy this script in production with real credentials or private keys.

