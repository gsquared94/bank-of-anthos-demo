# Using skaffold with a multi-repository application

## Introduction

### What is this project?
Forked from the original **[Bank of Anthos](https://github.com/GoogleCloudPlatform/bank-of-anthos)** project, it is a sample HTTP-based web app that simulates a bank's payment processing network, allowing users to create artificial bank accounts and complete transactions.

It consists of multiple microservices with each service code living in its own Github repository:
| Service(click to see repo)                                          | Language      | Description                                                                                                                                  |
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

### What you'll learn

- how to build and deploy a multi-repository application using `skaffold`
- how to iterate on individual services or groups of services.

___

**Time to complete**: <walkthrough-tutorial-duration duration=15></walkthrough-tutorial-duration>

Click the **Start** button to move to the next step.

## First steps

### Install Skaffold

Check the version of `skaffold` installed.

Run:
```bash
skaffold version
```

 If it is older than `v1.18.0`, then we manually install it. 

Run:
```bash
curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/v1.18.0/skaffold-linux-amd64 
sudo install skaffold /usr/bin/skaffold
```

### Start a Minikube cluster

We'll use `minikube` as our local kubernetes cluster of choice.

Run:
```bash
minikube start
```

### Fetch the required services repo

This repository lists all the constituent service repositories as git submodules. We'll fetch the latest code for each repository. 

Run:
```bash
git submodule update --init --force --recursive
```

<walkthrough-footnote>
    In an actual scenario, we'd most likely be interested in only a subset of services and would only clone them.
</walkthrough-footnote>

## Build and deploy all services

The root <walkthrough-editor-open-file filePath="skaffold.yaml">skaffold.yaml</walkthrough-editor-open-file> file lists all the services as dependencies using the `requires` stanza. Launching `skaffold` with this configuration will import all the specified dependencies as skaffold artifacts in a single session. Each service repository defines its own `skaffold.yaml` configuration (eg. <walkthrough-editor-open-file filePath="accounts-db/skaffold.yaml">accounts-db/skaffold.yaml</walkthrough-editor-open-file>) that instructs `skaffold` how to build and deploy it. 

Run:
```bash
skaffold dev --port-forward --tail=false
```

Once all images are built (might take a while the first time) and deployed (check for streaming log message `"Deployments stabilized in x.xx seconds"`) click on the <walkthrough-web-preview-icon></walkthrough-web-preview-icon> icon and select `Change Port` and change the preview port to `4503`. This should redirect to the frontend service and show the running application.

<walkthrough-footnote>
    `--port-forward` flag forwards local ports to all service ports.
    `--tail=false` supresses streaming container logs
</walkthrough-footnote>

## Develop only selected services

We can scope each `skaffold` session to only a subset of services by specifying the `-m` or `--module` flag.

Look at the <walkthrough-editor-open-file filePath="docs/architecture.png">architecture diagram</walkthrough-editor-open-file> for the services. The minimal set of services required to test the feature `"logging in to an existing user account"` are:
- <walkthrough-editor-open-file filePath="accounts-db/skaffold.yaml">accounts-db</walkthrough-editor-open-file>
- <walkthrough-editor-open-file filePath="userservice/skaffold.yaml">userservice</walkthrough-editor-open-file>
- <walkthrough-editor-open-file filePath="frontend/skaffold.yaml">frontend</walkthrough-editor-open-file>

Run:
```bash
skaffold dev -m setup -m frontend -m userservice -m accounts-db --port-forward --tail=false
```

Images should already be available in the `skaffold` cache. Once deployed click on the <walkthrough-web-preview-icon></walkthrough-web-preview-icon> icon and click `Preview on port 4503`. This should redirect to the frontend service and show the running application. You should be able to login to the application and all transactions should fail as we aren't running those services.

<walkthrough-footnote>
    The `setup` module needs to be included when running with any other module since it installs some prereq resources. 
</walkthrough-footnote>

## Congratulations

<walkthrough-conclusion-trophy></walkthrough-conclusion-trophy>

All done!

You now know how to use Skaffold to develop multi-repository applications.
