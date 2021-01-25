# Bank of Anthos

**Bank of Anthos** is a sample HTTP-based web app that simulates a bank's payment processing network, allowing users to create artificial bank accounts and complete transactions.

This fork modifies the original [Bank of Anthos]((https://github.com/GoogleCloudPlatform/bank-of-anthos)) into a multi-repository microservices design to demonstrate how to integrate with `skaffold`'s multi-config support.
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

[![Open in Cloud Shell](https://gstatic.com/cloudssh/images/open-btn.svg)](https://ssh.cloud.google.com/cloudshell/editor?cloudshell_git_repo=https://github.com/gsquared94/bank-of-anthos-demo&cloudshell_workspace=.&cloudshell_tutorial=README.md#Quickstart)


1. **[Create a Google Cloud Platform project](https://cloud.google.com/resource-manager/docs/creating-managing-projects#creating_a_project)** or use an existing project. Set the `PROJECT_ID` environment variable and ensure the Google Kubernetes Engine API is enabled.

```
PROJECT_ID=""
gcloud services enable container --project ${PROJECT_ID}
```

2. **Create a Kubernetes cluster.**   
* Create a GKE cluster 
```
ZONE=us-central1-b
gcloud beta container clusters create bank-of-anthos \
--project=${PROJECT_ID} --zone=${ZONE} \
--machine-type=e2-standard-2 --num-nodes=4 \
--enable-stackdriver-kubernetes --subnetwork=default \
--tags=bank-of-anthos --labels csm=
```
* Alternately, if using the Cloud Shell IDE create a minikube cluster
```
minikube start
```

3. **Clone this repository.**

```
git clone https://github.com/gsquared94/bank-of-anthos-demo.git
cd bank-of-anthos
```

4. **Clone all submodule projects**
```
git submodule update --init --force --remote
```

5. **Install skaffold [v1.18.0](https://github.com/GoogleContainerTools/skaffold/releases/tag/v1.18.0) or newer**

* Linux:
```
curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/v1.18.0/skaffold-linux-amd64 && chmod +x skaffold && sudo mv skaffold /usr/local/bin
```

* macOS:
```
curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/v1.18.0/skaffold-darwin-amd64 && chmod +x skaffold && sudo mv skaffold /usr/local/bin
```
6. **Deploy the sample app to the cluster.**

```
skaffold run
```
7. **Iterate on only selected services by passing the `-m` flag.**
```
skaffold dev -m frontend -m balancereader --port-forward
```