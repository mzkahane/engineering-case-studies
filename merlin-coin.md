# Merlin coin app and redemption handling

## Overview
Merlin Coins are promotional coin redemption system integrated with the PQCanvass platform. Users receive a physical coin with a QR code that contains a unique GUID, scan to visit a dedicated web app, log in with their PQCanvass credentials, and redeem the code for free AI credits appoed to their account. After redemption, they are seemlessly handed off into a logged-in PQCanvass session. The system spans three repos and three layers: The frontend, backend, and database.

## My Role
I built the entire feature end to end across all three layers
- **Database layer:** Designed the table schema for tracking the coins (GUIDs) and their status. wrote stored procedures for coin redeem, lookup, existence check, and used-status check. I added the coins to the purchasable items table so they could show up as transactions, and set up the appropriate role-based permissions.
- **Backend API:** Implemented the POST /merlincoin/redeem/:guid endpoint with the full middleware chain, including authentication, input validation, and content type. Also implemented the MerlinCoin model/query classes, structured logging, and an internal email alert system for when a coin is redeemed.
- **Frontend app:** Built the complete Preact/Redux single page app. This consisted of 5 components: MerlinCoinApp, LoginStep, AccountSelectStep, SuccessStep, and busy/error step. Also built the Redux state with 4 slices, 3 async thunk actions, webpack config for 4 levels of testing environments, LESS styling with dark mode support and responsive breakpoints, and a session handoff mechanism to PQCanvass via sessionStorage.
- Deployed across dev, test, burnin, and prod environments.

## Tech Stack
- Frontend:
  - Preact (aliased as React)
  - Redux + redux-thunk
  - LESS
  - Webpack
- Backend:
  - PHP (custom "Still" framework), token-based auth
- Database:
  - MYSQL tables, stored procedures (SPs), transactions, and role-based permissions
- Build/Deploy:
  - Webpack (4 environment configs)
  - GitLab CI
 
## The Problem
PQCanvass needed a way to distribute promotional credits to prospective or existing customers. The requirement was:
- Generate unique redemption codes that could be distributed via physical coins
- Let users redeem codes by scanning a unique QR code on the coin that led to a simple web interface
- Apply credits directly to their PQCanvass acount wallet
- Seamlessly redirect them into PQCanvass after redemption (already logged in)
- Track redemptions for auditing, statistics, and to prevent double-use
- Support expiration dates on codes
- Send internal alerts when codes are redeemed

## Solution
Database: 
- merlin_coins table wiht GUID, credit_amount, used flag, date_used, exipration_date, and foreign keys to transaction/account/user
- `sp_redeem_merlin_coin`: atomic procedure that validates the coin, creates a transaction record, adds it as a line item, finalizes the transaction, credits the wallet, and marks the coin used. Returns a structured result with success/failure reason codes.
- 3 supporting read procedures for lookkup, existence, and status checks
- Role based permissions for reads and writes

Backend (single endpoint):
- POST /merlincoin/redeem/:guid with middleware chain: auth → content type → input validation → user loading
- Verifies user has access to the target account before calling the SP
- Maps SP failure reasons to HTTP status codes
- Sends CoinRedemptionAlertEmail on success (non-blocking)
- Structured logging at every decision point

Frontend (3-step flow):
1. LoginStep: Email/password form with HTML5 validation → calls POST /auth/sign-in → extracts credentials from response headers
2. AccountSelectStep: Radio button list of user's accounts (filters out demo account IDs) → dispatches redeem thunk → calls POST /merlincoin/redeem/:guid
3. SuccessStep: Shows credits awarded → calls POST /auth/accounts/:id/select → populates sessionStorage with credentials in PQCanvass format → redirects to PQCanvass URL

If the user only has one account, step 2 is skipped entirely. URL parameter ?redeem= carries the code/GUID, ?source= for marketing department

## Challeneges
This was my first ever full stack project, so most of my time was spent learning as I went. I have no doubt it could be done better or cleaner, but I view it as a valuable learning experience that left me with a better understanding of full stack web development. Because of this, every step of the way felt like a new challenge. There weren't any steps that stood out as being particularly challenging, outside of figuring out what and why I was doing.

## Lessons Learned
- Moving validation from PHP to the stored procedure made redemption atomic and eliminated race conditions
- How to propagate error codes from the database through to the frontend, translated into meaningful error messages
- Building all three layers gave me end to end visibility into how a request flows from URL parameter to database transaction to UI confirmation.
