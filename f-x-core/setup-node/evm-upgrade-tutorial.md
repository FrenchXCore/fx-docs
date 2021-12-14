# EVM Upgrade Tutorial

### f(x)Core hard fork upgrade--support EVM compatibility

> This upgrade introduces a new module `evm` which will enable `ethereum` compatibility

### Upgrade steps

1. Ensure you have stopped the node❗

```
# with Daemon
sudo systemctl stop fxcored

# with PID
ps -ef | grep fxcored
kill -9 <PID>
```

&#x20;2\. Pulling the latest fx-core code base (ensure that you are in the fx-core folder) and be sure to checkout the `evm` branch:

```
git pull

# Testnet
git pull 

git checkout testnet-evm
```

&#x20;3\. Update fxcored (ensure that you are in the fx-core folder):

```
make go.sum

# testnet
make install-testnet

# mainnet
make install
```

&#x20;4\. Add EVM configuration (preferably adding it after the last line of the `app.toml` file) into the **`app.toml`** file there are multiple ways to do this, a few suggestions include opening the file in a [vi editor](https://www.cs.colostate.edu/helpdocs/vi.html) or editing it by remoting into the terminal using [Visual Studio Code](https://code.visualstudio.com/docs/remote/ssh):

```
cat >> ~/.fxcore/config/app.toml <<EOF

###############################################################################
###                             EVM Configuration                           ###
###############################################################################

[evm]

# Tracer defines the 'vm.Tracer' type that the EVM will use when the node is run in
# debug mode. To enable tracing use the '--trace' flag when starting your node.
# Valid types are: json|struct|access_list|markdown
tracer = ""

###############################################################################
###                           JSON RPC Configuration                        ###
###############################################################################

[json-rpc]

# Enable defines if the gRPC server should be enabled.
enable = true

# Address defines the EVM RPC HTTP server address to bind to.
address = "0.0.0.0:8545"

# Address defines the EVM WebSocket server address to bind to.
ws-address = "0.0.0.0:8546"

# API defines a list of JSON-RPC namespaces that should be enabled
# Example: "eth,txpool,personal,net,debug,web3"
api = "eth,net,web3"

# GasCap sets a cap on gas that can be used in eth_call/estimateGas (0=infinite). Default: 25,000,000.
gas-cap = 25000000

# EVMTimeout is the global timeout for eth_call. Default: 5s.
evm-timeout = "5s"

# TxFeeCap is the global tx-fee cap for send transaction. Default: 1eth.
txfee-cap = 1

# FilterCap sets the global cap for total number of filters that can be created
filter-cap = 200

# FeeHistoryCap sets the global cap for total number of blocks that can be fetched
feehistory-cap = 100


###############################################################################
###                             TLS Configuration                           ###
###############################################################################

[tls]

# Certificate path defines the cert.pem file path for the TLS configuration.
certificate-path = ""

# Key path defines the key.pem file path for the TLS configuration.
key-path = ""
EOF
```

&#x20;5\. Check fxcored environment & version

```
fxcored network
```

Return (you should now see a field that says "EvmSupportBlock"):

```
ChainId: dhobyghaut
CrossChainSupportBscBlock: "1"
CrossChainSupportPolygonBlock: "1"
CrossChainSupportTronBlock: "1"
EIP155ChainID: "90001"
EvmSupportBlock: "408000"
GravityPruneValsetsAndAttestationBlock: "1"
GravityValsetSlashBlock: "1"
Network: testnet
```

Cross reference the latest commit hash to the commit in our [official github page](https://github.com/FunctionX/fx-core):

```
fxcored version
```

Check evm configuration is added successfully:

```
fxcored config app.toml
```

Return (you should see an EVM configuration):

```
...
EVM:
  Tracer: ""
GRPC:
  Address: 0.0.0.0:9090
  Enable: true
HaltHeight: 0
HaltTime: 0
IndexEvents: []
InterBlockCache: true
JSONRPC:
  API:
  - eth
  - net
  - web3
  Address: 0.0.0.0:8545
  EVMTimeout: 5e+09
  Enable: true
  FeeHistoryCap: 100
  FilterCap: 200
  GasCap: 2.5e+07
  TxFeeCap: 1
  WsAddress: 0.0.0.0:8546
...
```

&#x20;6\. Features of client.toml

> Users can specify the configuration of certain commands in the configuration file `client.toml`
>
> Configure priority --flag> client.toml(default)
>
> A point to note is that client.toml configuration are for fxcored CLI, while app.toml and config.toml are configurations for the node.

| key               | default                 | Optional value         | description                                                                                                                 |
| ----------------- | ----------------------- | ---------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| `chain-id`        |                         | `fxcore`               | Chain ID-used when signing transactions                                                                                     |
| `keyring-backend` | `os`                    | `os`/`file`/`test`     | Keys storage method os: stored in the system password, file: file, specified password encryption, test: file, no encryption |
| `output`          | `text`                  | `text`/`json`          | Output format when querying                                                                                                 |
| `node`            | `tcp://localhost:26657` |                        | Node address to be called                                                                                                   |
| `broadcast-mode`  | `sync`                  | `sync`/`async`/`block` | Broadcast transaction mode                                                                                                  |

Modify client.toml command `fxcored config $key $value` (example):

```
fxcored config chain-id fxcore

fxcored config keyring-backend test

fxcored config output json

fxcored config node "tcp://127.0.0.1:26657"

fxcored config broadcast-mode block
```

&#x20;7\. Restart the node:

```
sudo systemctl restart fxcored
```

&#x20;8\. Check whether the node is participating in consensus:

```
cat $HOME/.fxcore/data/priv_validator_state.json
```

it should return something similar to the following:

```
{
  "height": "347450",
  "round": 0,
  "step": 3,
  "signature": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
  "signbytes": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
}
```

### Governance for upgrading:

1. The team will initiate a proposal for the upgrade (optional)
2. All nodes will have to complete upgrading by the stipulated block height
3. The nodes who have not upgraded by then will not be part of the consensus and if your validator node experiences [too long a downtime, it will be jailed and slashed](../../validators/validator-faq.md#what-are-the-slashing-conditions).
4. There will be a governance proposal initiated after to initialize and affirm this upgrade
5. After the proposal is passed, the EVM module will automatically start to run and the corresponding port 8545 will start. At this stage, the validator does not need to do anything except whether to vote in the proposal.
