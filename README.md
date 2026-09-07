# FieldForceConnect – Postman API Testing (Machine Test)

This repo contains the Postman **environment** and **collection** used to test the
Login and Add Customer APIs on `https://test.fieldforceconnect.com/`.

## Files

- `FieldForceConnect.postman_environment.json` – environment with `base_url`,
  `username`, `password`, `invalid_password`, `customer_name`, `customer_mobile`,
  `userId`, `companyId`, and `customerId` variables.
- `FieldForceConnect.postman_collection.json` – requests for Login (valid/invalid)
  and Add Customer, with test scripts that capture `userId` / `customerId`
  automatically for reuse in later requests.

## 1. Environment & Base URL

An environment named **FieldForceConnect - Test** was created with:

| Variable                          | Purpose                                                                                                  |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| `base_url`                        | `https://test.fieldforceconnect.com` — used as `{{base_url}}` in every request so the host is never hardcoded. |
| `username` / `password`           | Credentials from the self-signup, referenced in the login body.                                          |
| `invalid_password`                | A deliberately wrong password, used for the negative login test.                                         |
| `customer_name` / `customer_mobile` | Parametrized values used when creating a new customer, so the same request can create different customers by just changing these variables. |
| `userId` / `companyId`            | Left empty initially; populated automatically from the Login response's test script. |
| `customerId`                      | Left empty initially; populated automatically from the Add Customer response's test script. |

## 2. Finding the real Login / Add Customer endpoints

The site is a SPA, so the endpoints aren't visible from the URL alone. They were
found using the browser's Network tab (DevTools) while performing each action manually:

1. Signed up on `https://test.fieldforceconnect.com/` and generated a password.
2. Opened DevTools → **Network** tab → filtered by **Fetch/XHR**, with "Preserve log" enabled.
3. Logged in through the UI and captured the actual login request.
4. Clicked **Add Customer** on the dashboard, filled the form, and captured the actual save request.

### Findings

- **Login** calls `POST /api/account/authenticate` with a JSON body of
  `username` and `password`. The response does **not** return a Bearer token
  (the `token` field comes back empty) — instead the app relies on the
  session cookie set by the server, which Postman's cookie jar carries
  automatically to subsequent requests on the same domain. The response does
  include a `userId` and `companyId`, which are captured into environment
  variables for use elsewhere.
- **Add Customer** calls `POST /api/CRM/Lead`. Interestingly, "Customer" is
  not a separate entity in the backend — a Customer is stored as a **Lead**
  record (the payload/response use fields like `LeadName`, `LeadTypeId`,
  `LeadStageId`). Sending `LeadId: 0` creates a new record; a non-zero
  `LeadId` would update an existing one.
- **Get Customers** (list) calls `GET /api/CRM/Leads`.

## 3. Requests included

### Login – Valid Credentials (POST)
Sends `{{username}}` / `{{password}}` to `/api/account/authenticate`. Test
script asserts a 200 response, `success: true`, and that the returned user's
email matches `{{username}}`. It then stores `userId` and `companyId` into
the environment for reuse.

### Login – Invalid Credentials (POST)
Same endpoint, sent with `{{invalid_password}}`. Test script checks for
either an error status code (400/401/403) or a `success: false` in the body,
since the API can respond either way.

### Add Customer (POST)
Creates a customer (technically a Lead) using `{{customer_name}}` and
`{{customer_mobile}}` — changing these two variables is enough to create a
different customer, satisfying the parametrization requirement. Test script
asserts `Status: 200`, `Message: "Success"`, that a new `Id` was generated,
and that the returned `LeadName` matches what was sent. The new `Id` is
saved as `customerId` for later use.

### Get Customers (Leads list) (GET)
Supporting request to confirm the newly created customer appears in the list.

## 4. How to run

1. Import both JSON files into Postman (Environments + Collections).
2. Select the **FieldForceConnect - Test** environment.
3. Fill in `username` / `password` with your own signed-up credentials.
4. Run **Login – Valid Credentials** → confirms 200 + captures `userId`/`companyId`.
5. Run **Login – Invalid Credentials** → confirms rejection.
6. Run **Add Customer** → confirms the customer (Lead) is created; change
   `customer_name` / `customer_mobile` to create additional customers.
7. Run **Get Customers (Leads list)** to verify the new record appears.

## Notes

- Sensitive values (`password`, `invalid_password`) are stored with type
  `secret` in the environment file so they don't display in plain text in
  the Postman UI.
- Auth for this app is session/cookie-based rather than a Bearer token —
  running the Login request first in the same Postman session is required
  before Add Customer / Get Customers will succeed.
