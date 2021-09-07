### Cross-network transactions

For example, I used croeseid-testnet ([https://crypto.org/docs/getting-started/croeseid-testnet.html#pre-requisites](https://crypto.org/docs/getting-started/croeseid-testnet.html#pre-requisites))

# **A) Installation and configuration relay**

1. Install `1.0.0-rc1–152-g112205b` versin via github

```bash
git clone https://github.com/cosmos/relayer.git
cd relayer && make install
rly version
```

2. Initialize the relayer's configuration.

```bash
rly config init
```

3. Add relevant chain configurations to the relayer's configuration to `kichain_config.json`.

```bash
# kichain_config.json
{
  "chain-id": "kichain-t-4",
  "rpc-addr": "http://127.0.0.1:26657",
  "account-prefix": "tki",
  "gas-adjustment": 1.5,
  "gas-prices": "0.025utki",   
  "trusting-period": "48h"
}
```

Similarly, we create a file `croeseid_config.json` for the second network

```bash
# croeseid_config.json
{
  "chain-id": "testnet-croeseid-4",
  "rpc-addr": "http://127.0.0.1:26652",
  "account-prefix": "tcro",
  "gas-adjustment": 1.5,
  "gas-prices": "0.025basetcro",   
  "trusting-period": "48h"
}
```

```bash
rly chains add -f kichain_config.json
rly chains add -f croeseid_config.json
```

4. Either restore or create new keys for the relayer to use when signing and relaying transactions.

To add new key:
```bash
rly keys add kichain-t-4 <ki_wallet_name>
rly keys add testnet-croeseid-4 <tcro_wallet_name>
```

or restore:

```bash
rly keys restore kichain-t-4 <ki_wallet_name> "ki_mnemonic"
rly keys restore testnet-croeseid-4 <tcro_wallet_name> "tcro_mnemonic"
```

5) Assign the relayer chain-specific keys created or imported above to the specific chain's configuration.

```bash
rly chains edit kichain-t-4 key <ki_wallet_name>

rly chains edit testnet-croeseid-4 key <tcro_wallet_name>
```

6) Both relayer accounts, need to funded with tokens in order to successfully sign and relay transactions between the IBC-connected networks. How this occurs depends on the network, context and environment, e.g. local or test networks can use a faucet.

Foucet for croeseid https://crypto.org/faucet

7) Ensure both relayer accounts are funded by querying each.

```
rly q balance kichain-t-4
rly q balance testnet-croeseid-4
```

8) Now initialize the light clients on each network

```bash
rly light init kichain-t-4 -f
rly light init testnet-croeseid-4 -f
```

9) Try to generate a new path representing a client, connection, channel and a specific port between the two networks

```bash
rly paths generate kichain-t-4 testnet-croeseid-4 transfer --port=transfer
```

If the channel does not open, then go to the config with the command:

```bash
nano /root/.relayer/config/config.yaml
```

Change paths section to the text below:

```bash
paths:
  transfer:
    src:
      chain-id: kichain-t-4
      port-id: transfer
      order: UNORDERED
      version: ics20-1
    dst:
      chain-id: testnet-croeseid-4
      port-id: transfer
      order: UNORDERED
      version: ics20-1
    strategy:
      type: naive
```
initialize the light clients on each network again

```bash
rly light init kichain-t-4 -f
rly light init testnet-croeseid-4 -f
```

Run command 

```bash
rly tx link transfer -d
```

until you see the output of the command:

```bash
I[2021-09-06|19:36:28.725] ★ Channel created: [kichain-t-4]chan{channel-79}port{transfer} -> [testnet-croeseid-4]chan{channel-35}port{transfer}
```
After that, the network data will appear in the configuration file(`nano /root/.relayer/config/config.yaml`), for example, like mine
```bash
paths:
  transfer:
    src:
      chain-id: kichain-t-4
      client-id: 07-tendermint-11
      connection-id: connection-56
      channel-id: channel-79
      port-id: transfer
      order: UNORDERED
      version: ics20-1
    dst:
      chain-id: testnet-croeseid-4
      client-id: 07-tendermint-65
      connection-id: connection-40
      channel-id: channel-35
      port-id: transfer
      order: UNORDERED
      version: ics20-1
    strategy:
      type: naive
```
11) Check the created channel:

```bash
rly paths list -d
```

The output should be like this

```bash
0: transfer             -> chns(✔) clnts(✔) conn(✔) chan(✔) (kichain-t-4:transfer<>testnet-croeseid-4:transfer)
```

# **B) Make transaction**

1. To make transaction use this command

```bash
rly transact transfer [src-chain-id] [dst-chain-id] [amount] [dst-addr] [flags]
```

for example send the transaction from kichain to croeseid

```bash
rly tx transfer kichain-t-4 testnet-croeseid-4 1200utki tcro1emr3cgpw9lpgr9cc99cvdtv7yr5aeq4e5h0ss4 --path transfer -d
```

The output should be like this

```bash
I[2021-09-06|19:38:14.868] ✔ [kichain-t-4]@{220332} - msg(0:transfer) hash(549AB3EDB72AD77B0980FD43D39C4647A90775614D090A546FDCAB07AD1C9C23)
```

[https://ki.thecodes.dev/tx/549AB3EDB72AD77B0980FD43D39C4647A90775614D090A546FDCAB07AD1C9C23](https://ki.thecodes.dev/tx/549AB3EDB72AD77B0980FD43D39C4647A90775614D090A546FDCAB07AD1C9C23)

2. To send the transaction from croeseid to kichain

```bash
rly tx transfer testnet-croeseid-4 kichain-t-4 1000000basetcro tki1f602k2vlf3y8eayw5j2rrpkuy5h96gazy8ypx8 --path transfer -d
```

```bash
I[2021-09-06|19:39:08.612] ✔ [testnet-croeseid-4]@{300421} - msg(0:transfer) hash(24A4B55125DAFE37DB1462DA30957DDB9196C9D4215583DC529F2ED0172AFF47)
```

[https://crypto.org/explorer/croeseid4/tx/24A4B55125DAFE37DB1462DA30957DDB9196C9D4215583DC529F2ED0172AFF47](https://crypto.org/explorer/croeseid4/tx/24A4B55125DAFE37DB1462DA30957DDB9196C9D4215583DC529F2ED0172AFF47)

# **C) Create relayr client**
Create the service file 

```bash
sudo tee /etc/systemd/system/rlyd.service > /dev/null <<EOF
[Unit]
Description=relayer client
After=network-online.target, kichaind.service
[Service]
User=$USER
ExecStart=$(which rly) start transfer -d
Restart=always
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

```bash
sudo systemctl daemon-reload && sudo systemctl enable rlyd && sudo systemctl start rlyd
```

Knowing the parameters of the channel, you can send cross-network transactions

```bash
kid tx ibc-transfer transfer transfer channel-76 <tcro_wallet> 1200utki --from <ki_wallet_name>--fees=5000utki --gas=auto --chain-id kichain-t-4 --home $HOME/kichain/kid
```
