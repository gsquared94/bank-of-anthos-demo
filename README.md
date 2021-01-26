# Bank of Anthos

**Bank of Anthos** is a sample HTTP-based web app that simulates a bank's payment processing network, allowing users to create artificial bank accounts and complete transactions.

This fork modifies the original [Bank of Anthos](https://github.com/GoogleCloudPlatform/bank-of-anthos) into a multi-repository microservices design to demonstrate how to integrate with `skaffold`'s multi-config feature.
## Service Architecture

![Architecture Diagram](./docs/architecture.png)
| Service                                          | Language      | Description                                                                                                                                  |
| ------------------------------------------------ | ------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| [frontend](https://github.com/gsquared94/bank-of-anthos-frontend)                       | Python        | Exposes an HTTP server to serve the website. Contains login page, signup page, and home page.                                                |
| [ledger-writer](https://github.com/gsquared94/bank-of-anthos-ledgerwriter)              | Java          | Accepts and validates incoming transactions before writing them to the ledger.                                                               |
| [balance-reader](https://github.com/gsquared94/bank-of-anthos-balancereader)            | Java          | Provides efficient readable cache of user balances, as read from `ledger-db`.                                                                |
| [transaction-history](https://github.com/gsquared94/bank-of-anthos-transactionhistory)  | Java          | Provides efficient readable cache of past transactions, as read from `ledger-db`.                                                            |
| [ledger-db](https://github.com/gsquared94/bank-of-anthos-ledger-db)                     | PostgreSQL | Ledger of all transactions. Option to pre-populate with transactions for demo users.                                                         |
| [user-service](https://github.com/gsquared94/bank-of-anthos-userservice)                | Python        | Manages user accounts and authentication. Signs JWTs used for authentication by other services.                                              |
| [contacts](https://github.com/gsquared94/bank-of-anthos-contacts)                       | Python        | Stores list of other accounts associated with a user. Used for drop down in "Send Payment" and "Deposit" forms. |
| [accounts-db](https://github.com/gsquared94/bank-of-anthos-accounts)                 | PostgreSQL | Database for user accounts and associated data. Option to pre-populate with demo users.                                                      |
| [loadgenerator](https://github.com/gsquared94/bank-of-anthos-loadgenerator)             | Python/Locust | Continuously sends requests imitating users to the frontend. Periodically creates new accounts and simulates transactions between them.      |


## Quickstart

Follow the `Google Cloud Shell` tutorial:

[![Open in Cloud Shell](https://gstatic.com/cloudssh/images/open-btn.svg)](https://ssh.cloud.google.com/cloudshell/editor?cloudshell_git_repo=https://github.com/gsquared94/bank-of-anthos-demo&cloudshell_workspace=.&cloudshell_tutorial=tutorial.md)

Alternately,

1. **Setup a Kubernetes cluster of choice ([GKE](https://cloud.google.com/kubernetes-engine), [minikube](https://minikube.sigs.k8s.io/docs/start/), [kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation), etc.)** 

2. **Clone this repository.**

```
git clone https://github.com/gsquared94/bank-of-anthos-demo.git
cd bank-of-anthos-demo
```

3. **Clone all submodule projects**

```
git submodule update --init --force --recursive
```

4. **Install skaffold [v1.18.0](https://github.com/GoogleContainerTools/skaffold/releases/tag/v1.18.0) or newer**

* Linux:

```
curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/v1.18.0/skaffold-linux-amd64 && chmod +x skaffold && sudo mv skaffold /usr/local/bin
```

* macOS:

```
curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/v1.18.0/skaffold-darwin-amd64 && chmod +x skaffold && sudo mv skaffold /usr/local/bin
```

5. **Deploy the sample app to the cluster.**

```
skaffold run
```

6. **Iterate on only selected services by passing the `-m` flag.**

```
skaffold dev -m frontend -m balancereader --port-forward
```