# bank-of-anthos

## Step 1 - Workshop bank-of-anthos-oa
Dynatrace OneAgent instrumentation  

## Step 2 - Workshop bank-of-anthos-otel
Opentelemetry instrumentation  

## Step 3 - Workshop bank-of-anthos
Dynatrace OneAgent instrumentation and Opentelemetry instrumentation


## Screenshots

| Sign in                                                                                                        | Home                                                                                                    |
| ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| [![Login](/img/login.png)](/img/login.png) | [![User Transactions](/img/transactions.png)](/img/transactions.png) |


## Service architecture

![Architecture Diagram](/img/architecture.png)

| Service                                                 | Language      | Description                                                                                                                                  |
| ------------------------------------------------------- | ------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| [frontend](/kubernetes-manifests/frontend.yaml)                              | Python        | Exposes an HTTP server to serve the website. Contains login page, signup page, and home page.                                                |
| [ledger-writer](/kubernetes-manifests/ledgerwriter.yaml)              | Java          | Accepts and validates incoming transactions before writing them to the ledger.                                                               |
| [balance-reader](/kubernetes-manifests/balance-reader.yaml)            | Java          | Provides efficient readable cache of user balances, as read from `ledger-db`.                                                                |
| [transaction-history](/kubernetes-manifests/transaction-history.yaml)  | Java          | Provides efficient readable cache of past transactions, as read from `ledger-db`.                                                            |
| [ledger-db](/kubernetes-manifests/ledger-db.yaml)                     | PostgreSQL    | Ledger of all transactions. Option to pre-populate with transactions for demo users.                                                         |
| [user-service](/kubernetes-manifests/userservice.yaml)              | Python        | Manages user accounts and authentication. Signs JWTs used for authentication by other services.                                              |
| [contacts](/kubernetes-manifests/contacts.yaml)                     | Python        | Stores list of other accounts associated with a user. Used for drop down in "Send Payment" and "Deposit" forms.                              |
| [accounts-db](/kubernetes-manifests/accounts-db.yaml)               | PostgreSQL    | Database for user accounts and associated data. Option to pre-populate with demo users.                                                      |
| [loadgenerator](/kubernetes-manifests/loadgenerator.yaml)                    | Python/Locust | Continuously sends requests imitating users to the frontend. Periodically creates new accounts and simulates transactions between them.      |