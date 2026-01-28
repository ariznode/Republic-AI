# Republic-AI

## Spesification

Recommended : 

- CPU : 6 core
- RAM : 16 GB
- Storage : 500 GB

##Republic AI Validator Guide

### Install Depedency

```bash
sudo apt update && sudo apt upgrade -y
```

```bash
sudo apt install screen curl iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

### Install Republicd Binary

```bash
VERSION="v0.1.0"
curl -L "https://media.githubusercontent.com/media/RepublicAI/networks/main/testnet/releases/${VERSION}/republicd-linux-amd64" -o /tmp/republicd
chmod +x /tmp/republicd
sudo mv /tmp/republicd /usr/local/bin/republic
```

### Initialize Node

```bash
MONIKER="YourName"
```

Note : change "YourName" with your name or anything

```bash
republicd init $MONIKER --chain-id raitestnet_77701-1 --home "$HOME/.republicd"
```

```bash
curl -s https://raw.githubusercontent.com/RepublicAI/networks/main/testnet/genesis.json > "$HOME/.republicd/config/genesis.json"
```

### Configure State Sync

```bash
REPUBLIC_HOME="$HOME/.republicd"
SNAP_RPC="https://statesync.republicai.io"
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
PEERS="e281dc6e4ebf5e32fb7e6c4a111c06f02a1d4d62@3.92.139.74:26656,cfb2cb90a241f7e1c076a43954f0ee6d42794d04@54.173.6.183:26656,dc254b98cebd6383ed8cf2e766557e3d240100a9@54.227.57.160:26656"
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" "$REPUBLIC_HOME/config/config.toml"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" "$REPUBLIC_HOME/config/config.toml"
```

### Create Systemd Service

```bash
sudo tee /etc/systemd/system/republicd.service > /dev/null <<EOF
[Unit]
Description=Republic Protocol Node
After=network-online.target

[Service]
User=$USER
ExecStart=/usr/local/bin/republicd start --home $HOME/.republicd --chain-id raitestnet_77701-1
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

6. Start Node

```bash
sudo systemctl daemon-reload
sudo systemctl enable republicd
sudo systemctl start republic
```

### Check node logs

```bash
journalctl -u republicd -f -o cat
```

### Check Status

```bash
republicd status | jq '.sync_info'
```

### Create Wallet

```bash
republicd keys add mywallet
```

Note : change 'mywallet' to your name or anything

### Fund your wallet

- Join Discord : https://discord.gg/republicai
- Go to #faucet
- Paste your validator wallet


### Check Balance

```bash
republicd query bank balances youraddress
```

Note : change youraddress

### Register Validator

```bash
republicd tx staking create-validator \
  --amount=1000000000000000000000arai \
  --pubkey=$(republicd comet show-validator) \
  --moniker="YourMoniker" \
  --chain-id=raitestnet_77701-1 \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --gas=auto \
  --gas-adjustment=1.5 \
  --gas-prices="250000000arai" \
  --from=YourWallet \
  -y
```

Note : Change YourMoniker with your moniker name on prev steps
Note : Change YourWallet with your wallet name on prev steps

### Check Validator Status

```bash
republicd query staking validator $(republicd keys show WalletName --bech val -a)
```

Note : chgange WalletName with your wallet name from prev steps

### Back up wallet

```bash
cat ~/.republicd/config/priv_validator_key.json
```
