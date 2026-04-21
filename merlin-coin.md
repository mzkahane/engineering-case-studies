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
  - MYSQL tables, stored procedures, transactions, and role-based permissions
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
