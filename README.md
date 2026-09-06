# FieldForceConnect – Postman API Testing (Machine Test)

This repo contains the Postman **environment** and **collection** used to test the
Login and Add Customer APIs on `https://test.fieldforceconnect.com/`.

## Files
- `FieldForceConnect.postman_environment.json` – environment with `base_url`,
  `email`, `password`, `invalid_password`, and `auth_token` variables.
- `FieldForceConnect.postman_collection.json` – requests for Login (valid/invalid)
  and Add Customer, with test scripts that capture the auth token automatically.

## 1. Environment & Base URL
An environment named **FieldForceConnect - Test** was created with:
| Variable | Purpose |
|---|---|
| `base_url` | `https://test.fieldforceconnect.com` — used as `{{base_url}}` in every request so the host is never hardcoded. |
| `email` / `password` | Credentials from the self-signup, referenced as `{{email}}` / `{{password}}` in the login body. |
| `invalid_password` | A deliberately wrong password, used for the negative login test. |
| `auth_token` | Left empty initially; populated automatically by the Login request's test script and reused via `{{auth_token}}` in the Authorization header of the Add Customer request. |

## 2. Finding the real Login / Add Customer endpoints
The site is a SPA, so the endpoints aren't guessable from the URL alone — they're
found via the browser's network inspector:

1. Sign up on `https://test.fieldforceconnect.com/` with your email and generate a password.
2. Open Chrome/Edge DevTools → **Network** tab → filter by **Fetch/XHR**.
3. Log out (if needed) and log back in through the UI. Watch for the POST request
   fired when you submit the login form — note its **URL**, **request payload**,
   and **response body** (this is where the auth token/cookie comes from).
4. Update the `Login` requests in the collection: replace `/api/auth/login` with
   the real path, and adjust the JSON body keys to match what the UI actually sends.
5. In the collection's Test script, adjust `jsonData.token` (or whatever field
   holds the token/session id in the real response) so `auth_token` gets set correctly.
6. Repeat the same DevTools capture while clicking **Add Customer** on the dashboard
   to get the real endpoint, method, headers (Bearer token vs. cookie vs. custom
   header), and the exact field names the form submits.

## 3. Requests included
### Login – Valid Credentials (POST)
Sends `{{email}}` / `{{password}}` to the login endpoint. Test script asserts a
200 response and stores the returned token into `{{auth_token}}` for reuse.

### Login – Invalid Credentials (POST)
Same endpoint, sent with `{{invalid_password}}`. Test script asserts the API
returns a 400/401/403 with an error message, confirming invalid credentials are rejected.

### Add Customer (POST)
Uses `Authorization: Bearer {{auth_token}}` (adjust if the real API uses a
different auth scheme) to create a customer with `name`, `email`, `phone`.
Test script asserts a 200/201 and a `data` object in the response.

### Get Customers (GET, example)
A supporting GET request to confirm the newly created customer appears in the list.

## 4. How to run
1. Import both JSON files into Postman (Environments + Collections).
2. Select the **FieldForceConnect - Test** environment.
3. Fill in `email` / `password` with your signed-up credentials.
4. Run **Login – Valid Credentials** → confirms 200 + captures `auth_token`.
5. Run **Login – Invalid Credentials** → confirms rejection.
6. Run **Add Customer** → confirms the customer is created using the captured token.
7. (Optional) Run **Get Customers** to verify the new record.

## Notes
- Placeholder paths (`/api/auth/login`, `/api/customers`) and field names must be
  replaced with the real values captured from the browser Network tab before running.
- Sensitive values (password, token) are stored with type `secret` in the
  environment file so they don't display in plain text in the Postman UI.
