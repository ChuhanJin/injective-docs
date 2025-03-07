# Peggo

If you're on this page then you've probably become a Validator on Injective. Congratulations! Configuring `peggo` is the final step of your setup.

Example of `.env` for peggo:

```bash
PEGGO_ENV="local"         # environment name for metrics (dev/test/staging/prod/local)
PEGGO_LOG_LEVEL="debug"   # log level depth

PEGGO_COSMOS_CHAIN_ID="injective-1"           # chain ID of the Injective network
PEGGO_COSMOS_GRPC="tcp://localhost:9090"      # gRPC of your injectived process
PEGGO_TENDERMINT_RPC="http://localhost:26657" # Tendermint RPC of your injectived process

# Note: omitting PEGGO_COSMOS_GRPC and PEGGO_TENDERMINT_RPC enables stand-alone peggo mode. In this mode,
# peggo is connected to load balanced endpoints provided by the Injective network. This decouples peggo's connection from your injectived process.

# Injective config
PEGGO_COSMOS_FEE_DENOM="inj"            # token used to pay fees on Injective
PEGGO_COSMOS_GAS_PRICES="160000000inj"  # default --gas-prices flag value for sending messages to Injective
PEGGO_COSMOS_KEYRING="file"             # keyring backends ("os", "file", "kwallet", "memory", "pass", "test")
PEGGO_COSMOS_KEYRING_DIR=               # path to your keyring dir
PEGGO_COSMOS_KEYRING_APP="peggo"        # arbitrary name for your keyring app
PEGGO_COSMOS_FROM=                      # account address of your Validator (or your Delegated Orchestrator)
PEGGO_COSMOS_FROM_PASSPHRASE=           # keyring passphrase
PEGGO_COSMOS_PK=                        # private key of your Validator (or your Delegated Orchestrator)
PEGGO_COSMOS_USE_LEDGER=false

# Ethereum config
PEGGO_ETH_KEYSTORE_DIR=               # path to your Ethereum keystore
PEGGO_ETH_FROM=                       # your Ethereum address (must be Delegated Ethereum address if you're a Validator)
PEGGO_ETH_PASSPHRASE=                 # passphrase of your Ethereum keystore
PEGGO_ETH_PK=                         # private key of your Ethereum address
PEGGO_ETH_GAS_PRICE_ADJUSTMENT=1.3    # suggested Ethereum gas price will be adjusted by this factor (Relayer)
PEGGO_ETH_MAX_GAS_PRICE="500gwei"     # max gas price allowed for sending Eth transactions (Relayer)
PEGGO_ETH_CHAIN_ID=1                  # chain ID of Ethereum network
PEGGO_ETH_RPC="http://localhost:8545" # RPC of your Ethereum node
PEGGO_ETH_ALCHEMY_WS=""               # optional websocket endpoint for listening pending transactions on Peggy.sol
PEGGO_ETH_USE_LEDGER=false 

# Price feed provider for token assets (Batch Creator)
PEGGO_COINGECKO_API="https://api.coingecko.com/api/v3"

# Relayer config
PEGGO_RELAY_VALSETS=true                      # set to `true` to relay Validator Sets
PEGGO_RELAY_VALSET_OFFSET_DUR="5m"            # duration which needs to expire before a Valset is eligible for relaying 
PEGGO_RELAY_BATCHES=true                      # set to `true` to relay Token Batches
PEGGO_RELAY_BATCH_OFFSET_DUR="5m"             # duration which needs to expire before a Token Batch is eligible for relaying
PEGGO_RELAY_PENDING_TX_WAIT_DURATION="20m"    # time to wait until a pending tx is processed

# Batch Creator config
PEGGO_MIN_BATCH_FEE_USD=23.2  # minimum amount of fee a Token Batch must satisfy to be created

# Metrics config
PEGGO_STATSD_PREFIX="peggo."
PEGGO_STATSD_ADDR="localhost:8125"
PEGGO_STATSD_STUCK_DUR="5m"
PEGGO_STATSD_MOCKING=false
PEGGO_STATSD_DISABLED=true
```

{% hint style="info" %}
**IMPORTANT NOTE:** if you're running your own `injectived` (Injective node) and `geth` (Ethereum node) processes, ensure that they are in sync with the latest state. Outdated nodes can skew the business logic of `peggo` to display "false alarm" logs sometimes.
{% endhint %}

## Step 1: Configuring .env

```bash
# official Injective mainnet .env config 
mkdir ~/.peggo
cp mainnet-config/10001/peggo-config.env ~/.peggo/.env
cd ~/.peggo
```

Ethereum config

First, update the `PEGGO_ETH_RPC` in the `.env` file with a valid Ethereum EVM RPC Endpoint.

To set up your own Ethereum full node, follow the instructions [here](https://ethereum.org/en/developers/docs/nodes-and-clients/run-a-node/). It's possible to use an external Ethereum RPC provider such as Alchemy or Infura, but keep in mind that the Peggo bridge relayer makes a heavy use of `eth_getLogs` calls which may increase your cost burden, depending on your provider.

## **Managing Ethereum keys for `peggo`**

Peggo supports two options to provide signing key credentials - using the Geth keystore (recommended) or by providing a plaintext Ethereum private key.

#### **Option 1. Geth Keystore**

You can find instructions for securely creating a new Ethereum account using a keystore in the Geth Documentation [here](https://geth.ethereum.org/docs/interface/managing-your-accounts).

For convenience, an example is provided below.

```bash
geth account new --datadir=/home/ec2-user/.peggo/data/

INFO [03-23|18:18:36.407] Maximum peer count                       ETH=50 LES=0 total=50
Your new account is locked with a password. Please give a password. Do not forget this password.
Password:
Repeat password:

Your new key was generated

Public address of the key:   0x9782dc957DaE6aDc394294954B27e2118D05176C
Path of the secret key file: /home/ec2-user/.peggo/data/keystore/UTC--2021-03-23T15-18-44.284118000Z--9782dc957dae6adc394294954b27e2118d05176c

- You can share your public address with anyone. Others need it to interact with you.
- You must NEVER share the secret key with anyone! The key controls access to your funds!
- You must BACKUP your key file! Without the key, it's impossible to access account funds!
- You must REMEMBER your password! Without the password, it's impossible to decrypt the key!
```

Make sure you heed the warnings that geth provides, particularly in backing up your key file so that you don't lose your keys by mistake. We also recommend not using any quote or backtick characters in your passphrase for peggo compatibility purposes.

You should now set the following env variables:

```bash
# example values, replace with your own
PEGGO_ETH_KEYSTORE_DIR=/home/ec2-user/.peggo/data/keystore
PEGGO_ETH_FROM=0x9782dc957DaE6aDc394294954B27e2118D05176C
PEGGO_ETH_PASSPHRASE=12345678
```

Then ensure that your Ethereum address has enough ETH.

#### **Option 2. Ethereum Private Key (Unsafe)**

Simply update the `PEGGO_ETH_PK` with a new Ethereum Private Key from a new account.

Then ensure that your Ethereum address has enough ETH.

## Injective config

### **Creating your delegated Cosmos Key for sending Injective transactions**

Your peggo orchestrator can either:

* Use an explicitly delegated account key specific for sending validator specific Peggy transactions (i.e., `ValsetConfirm`, `BatchConfirm`, and `SendToCosmos` transactions) or
* Simply use your validator's account key ("your Validator is your Orchestrator")

For isolation purposes, we recommend creating a delegated Cosmos key to send Injective transactions instead of using your validator account key.

To create a new key, run

```bash
injectived keys add $ORCHESTRATOR_KEY_NAME
```

Then ensure that your orchestrator inj address has INJ balance in it, so peggo orchestrator can send messages to Injective.

To obtain your orchestrator's inj address, run

```bash
injectived keys list $ORCHESTRATOR_KEY_NAME
```

You can transfer INJ from your validator account to orchestrator address using this command

```bash
injectived tx bank send $VALIDATOR_KEY_NAME  $ORCHESTRATOR_INJ_ADDRESS <amount-in-inj> --chain-id=injective-1 --keyring-backend=file --yes --node=tcp://localhost:26657 --gas-prices=500000000inj
```

Example

```bash
injectived tx bank send genesis inj1u3eyz8nkvym0p42h79aqgf37gckf7szreacy9e 20000000000000000000inj --chain-id=injective-1  --keyring-backend=file --yes --node=tcp://localhost:26657 --gas-prices=500000000inj
```

You can then verify that your orchestrator account has INJ balances by running

```bash
injectived q bank balances $ORCHESTRATOR_INJ_ADDRESS
```

### **Managing Cosmos account keys for `peggo`**

Peggo supports two options to provide Cosmos signing key credentials - using the Cosmos keyring (recommended) or by providing a plaintext private key.

#### **Option 1. Cosmos Keyring**

In the `.env` file, first specify the `PEGGO_COSMOS_FROM` and `PEGGO_COSMOS_FROM_PASSPHRASE` corresponding to your peggo account signing key.

If you are using a delegated account key configuration as recommended above, this will be your `$ORCHESTRATOR_KEY_NAME` and passphrase respectively. Otherwise, this should be your `$VALIDATOR_KEY_NAME` and associated validator passphrase.

Please note that the default keyring backend is `file` and that as such peggo will try to locate keys on disk by default.

To use the default injectived key configuration, you should set the keyring path to the home directory of your injectived node, e.g., `~/.injectived`.

You can also read more about the Cosmos Keyring setup [here](https://docs.cosmos.network/v0.46/run-node/keyring.html).

#### **Option 2. Cosmos Private Key (Unsafe)**

In the `.env` file, specify the `PEGGO_COSMOS_PK` corresponding to your peggo account signing key.

If you are using a delegated account key configuration as recommended above, this will be your orchestrator account's private key. Otherwise, this should be your validator's account private key.

To obtain your orchestrator's Cosmos private key (if applicable), run

```bash
injectived keys unsafe-export-eth-key $ORCHESTRATOR_KEY_NAME
```

To obtain your validator's Cosmos private key (if applicable), run

```bash
injectived keys unsafe-export-eth-key $VALIDATOR_KEY_NAME
```

Again, this method is less secure and is not recommended.

### Step 2: Register Your Orchestrator and Ethereum Address

You can register orchestrator and ethereum address only once. It **CANNOT** be updated later. So Check twice before running below command.

```bash
injectived tx peggy set-orchestrator-address $VALIDATOR_INJ_ADDRESS $ORCHESTRATOR_INJ_ADDRESS $ETHEREUM_ADDRESS --from $VALIDATOR_KEY_NAME --chain-id=injective-1 --keyring-backend=file --yes --node=tcp://localhost:26657 --gas-prices=500000000inj

```

* To obtain your validator's inj address, run, `injectived keys list $VALIDATOR_KEY_NAME`
* To obtain your orchestrators's inj address, `injectived keys list $ORCHESTRATOR_KEY_NAME`

Example:

```bash
injectived tx peggy set-orchestrator-address inj10m247khat0esnl0x66vu9mhlanfftnvww67j9n inj1x7kvxlz2epqx3hpq6v8j8w859t29pgca4z92l2 0xf79D16a79130a07e77eE36e8067AeA783aBdA3b6 --from validator-key-name --chain-id=injective-1 --keyring-backend=file --yes --node=tcp://localhost:26657 --gas-prices=500000000inj
```

You can verify successful registration by checking for your Validator's mapped Ethereum address on https://lcd.injective.network/peggy/v1/valset/current.

{% hint style="info" %}
**NOTE:** Once you've registered your Orchestrator with the `set-orchestrator-address` message, you **CANNOT** register again. Once this step is complete, your `Validator` is bound to the provided Ethereum address (as well the Delegated address you may have provided). In other words, your peggo must always run with the addresses you provided for registration.
{% endhint %}

### Step 3: Start the Relayer

```bash
cd ~/.peggo
peggo orchestrator
```

This starts the Peggo bridge (relayer / orchestrator).

### Step 4: Create a Peggo systemd service

Add `peggo.service` file with below content under `/etc/systemd/system/peggo.service`

```ini
[Unit]
  Description=peggo

[Service]
  WorkingDirectory=/home/ec2-user/.peggo
  ExecStart=/bin/bash -c 'peggo orchestrator '
  Type=simple
  Restart=always
  RestartSec=1
  User=ec2-user

[Install]
  WantedBy=multi-user.target
```

Then use the following commands to configure Environment variables, start and stop the peggo relayer.

```bash
sudo systemctl start peggo
sudo systemctl stop peggo
sudo systemctl restart peggo
sudo systemctl status peggo

# enable start on system boot
sudo systemctl enable peggo

# To check Logs
journalctl -f -u peggo
```

### Step 5: (Optional) Protect Cosmos Keyring from unauthorized access

{% hint style="info" %}
This is an advanced DevOps topic, consult with your sysadmin.
{% endhint %}

Learn more about Cosmos Keyring setup [here](https://docs.cosmos.network/v0.46/run-node/keyring.html). Once you've launched your node, the default keyring will have the validator operator key stored on disk in the encrypted form. Usually the keyring is located within node's homedir, i.e. `~/.injectived/keyring-file`.

Some sections of the Injective Staking documentation will guide you through using this key for governance purposes, i.e. submitting transactions and setting up an Ethereum bridge. In order to protect the keys from unauthorized access, even when the keyring passphrase is leaked via configs, you can set OS permissions to allow disk access to `injectived` / `peggo` processes only.

In Linux systems like Debian, Ubuntu and RHEL, this can be achieved using POSIX Access Control Lists (ACLs). Before beginning to work with ACLs, the file system must be mounted with ACLs turned on. There are some official guides for each distro:

* [Ubuntu](https://help.ubuntu.com/community/FilePermissionsACLs)
* [Debian](https://wiki.debian.org/Permissions)
* [Amazon Linux (RHEL)](https://www.redhat.com/sysadmin/linux-access-control-lists)

### Contribute

If you'd like to inspect the Peggo orchestrator source code and contribute, you can do so at [https://github.com/InjectiveLabs/peggo](https://github.com/InjectiveLabs/peggo).
