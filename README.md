# IOTA Sponsoring Service

Our submission for the [IOTA Foundation's Moveathon](https://www.moveathon.build/europe) Europe edition.

This repository contains the complete, runnable project, which combines the backend and frontend using Git submodules.

## The Team

- [Donovan Roubos](https://github.com/donovanroubos) (https://yummygum.com/)
- [Daniel Mader](https://github.com/daniel-mader) (https://www.impierce.com/)
- [Teun Kelting](https://github.com/Steun) (https://yummygum.com/)
- [Nander Stabel](https://github.com/nanderstabel) (https://www.impierce.com/)
- Nick Schenkel (https://yummygum.com/)

## Project Overview

The IOTA Gas Station is a powerful, "headless" API component that allows developers to sponsor transactions, removing a major friction point for user onboarding. However, it lacks a good user interface and additional features for managing clients and monitoring sponsored transactions.

We built the missing management and authorization layer for the IOTA Gas Station. Our project provides a polished, professional UI and a robust Rust backend that gives operators full, real-time control over their sponsorship activities.

**Key Features**

- Professional UI: A clean, responsive React frontend (from the Yummygum repository) that provides a dashboard for managing all Gas Station activities.

- Client & Group Management: Create "Clients" (end-users) and "Groups" to assign and manage virtual budgets.

- Pre-Transaction Authorization: We use the Gas Station's HTTP Hook feature to check our application's virtual balance before a transaction is sponsored. If a client is out of funds, the transaction is denied, preventing wallet draining.

- Real-Time Reconciliation: We use an event-streaming pipeline (Vector) to parse the Gas Station's transaction logs. When a transaction is successfully paid, the client's virtual balance is instantly updated.

- Analytics Dashboard: The UI includes graphs for historical spending, allowing operators to track sponsorship costs per client or group over time.

## Getting Started

### Prerequisites

- [Docker](https://www.docker.com/get-started/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- [Rust](https://www.rust-lang.org/tools/install)
- [Cargo](https://doc.rust-lang.org/cargo/getting-started/installation.html)
- Git
- cURL

**1. Clone the repository**

This repository uses Git submodules. Clone it recursively to pull in the frontend and backend dependencies. The corresponding repositories are:

- [IOTA Sponsoring Service Backend](https://github.com/impierce/iota-sponsoring-service/tree/feat/init-graphql)
- [IOTA Sponsoring Service UI](https://github.com/Yummygum/iot-gasstation-app)

```bash
git clone https://github.com/impierce/moveathon-iota-sponsoring-service
cd moveathon-iota-sponsoring-service
git submodule update --init --recursive
```

**2. Build the Gas Station tool**

The IOTA Gas Station (which is included as a submodule within our backend) provides a tool for generating a configuration. We need to build this tool first.

```bash
cargo build --manifest-path iota-sponsoring-service/submodules/gas-station/Cargo.toml
```

**3. Generate and Fund the Gas Station**

Now, we'll generate the configuration and fund the new wallet.

```bash
# 1. Generate the config file
iota-sponsoring-service/submodules/gas-station/target/debug/tool generate-sample-config \
 --docker-compose \
 --config-path config.yaml \
 --network testnet
```

This will generate a `config.yaml` file and the Gas Station's new IOTA address.

```bash
# 2. Fund the new wallet
# Use the address generated from the previous command
curl --location --request POST 'https://faucet.testnet.iota.cafe/gas' --header 'Content-Type: application/json' --data-raw '{
  "FixedAmountRequest": {
       "recipient": "<GENERATED_ADDRESS>"
  }
}'
```

**4. Configure the Application**

Before running, we need to make two small edits to the `config.yaml` file that was just created.

1. Update Balances: Change target-init-balance to 5000000 and refresh-interval-sec to 30.

```yaml
coin-init-config:
  target-init-balance: 5000000
  refresh-interval-sec: 30
```

2. Set the Authorization Hook: Update the `access-controller` section to point to our backend service. This is the crucial step that enables our pre-transaction authorization.

```yaml
access-controller:
  access-policy: deny-all
  rules:
    - action: "http://iota-sponsoring-service:8000/webhook/authorize-transaction"
```

**5. Run the Services**

With the configuration in place, you can now build and run the entire stack using Docker Compose.

```bash
docker compose up --build -d
```

You're all set! The frontend will be available at `http://localhost:3000`.
