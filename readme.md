<h1 align="center"> 0G

![image](https://0g.ai/media-kit/0G-Logo.png)


</h1>

** System Requirements**

| Component | Minimum Requirements |
|---|---|
| CPU | 4 |
| RAM | 8+ GB |
| Storage | 400 GB SSD |

** Required Installations**

```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

** Go Installation**

```
cd $HOME
VER="1.21.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

** Downloading Files**

```
git clone https://github.com/0glabs/0g-evmos.git
cd 0g-evmos
git checkout v1.0.0-testnet
make build
```
```
mkdir -p $HOME/.evmosd/cosmovisor/genesis/bin
mv build/evmosd $HOME/.evmosd/cosmovisor/genesis/bin/
rm -rf build
```

** System Link**

```
sudo ln -s $HOME/.evmosd/cosmovisor/genesis $HOME/.evmosd/cosmovisor/current -f
sudo ln -s $HOME/.evmosd/cosmovisor/current/bin/evmosd /usr/local/bin/evmosd -f
```

** Downloading Cosmovisor**

```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.5.0
```

** Create Service**

```
sudo tee /etc/systemd/system/evmosd.service > /dev/null << EOF
[Unit]
Description=evmosd node service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.evmosd"
Environment="DAEMON_NAME=evmosd"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.evmosd/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload
sudo systemctl enable evmosd.service
```

** Node Configuration**

```
evmosd config chain-id zgtendermint_9000-1
evmosd config keyring-backend os
evmosd config node tcp://localhost:16457
```

** Init**

**NOTE:** Enter your node name.

```
evmosd init NODE_NAME_HERE --chain-id zgtendermint_9000-1
```

** Genesis Addrbook**

```
curl -Ls https://github.com/0glabs/0g-evmos/releases/download/v1.0.0-testnet/genesis.json > $HOME/.evmosd/config/genesis.json
```

** Seed**

```
PEERS="1248487ea585730cdf5d3c32e0c2a43ad0cda973@peer-zero-gravity-testnet.trusted-point.com:26326" && \
SEEDS="8c01665f88896bca44e8902a30e4278bed08033f@54.241.167.190:26656,b288e8b37f4b0dbd9a03e8ce926cd9c801aacf27



**Gas Settings:**

```bash
sed -i "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.00252aevmos\"/" $HOME/.evmosd/config/app.toml
```

**Port Settings:**

```bash
echo "export G_PORT="16"" >> $HOME/.bash_profile
source $HOME/.bash_profile

sed -i.bak -e "s%:1317%:${G_PORT}317%g;
s%:8080%:${G_PORT}080%g;
s%:9090%:${G_PORT}090%g;
s%:9091%:${G_PORT}091%g;
s%:8545%:${G_PORT}545%g;
s%:8546%:${G_PORT}546%g;
s%:6065%:${G_PORT}065%g" $HOME/.evmosd/config/app.toml

sed -i.bak -e "s%:26658%:${CROSSFI_PORT}658%g;
s%:26657%:${G_PORT}657%g;
s%:6060%:${G_PORT}060%g;
s%:26656%:${G_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${G_PORT}656\"%;
s%:26660%:${G_PORT}660%g" $HOME/.evmosd/config/config.toml
```

**Snap:**

```bash
sudo apt install liblz4-tool

systemctl stop evmosd

cp $HOME/.evmosd/data/priv_validator_state.json $HOME/.evmosd/priv_validator_state.json.backup

cp $HOME/.evmosd/config/priv_validator_key.json $HOME/.evmosd/priv_validator_key.json.backup

evmosd tendermint unsafe-reset-all --home $HOME/.evmosd --keep-addr-book

curl -L http://37.120.189.81/0g_testnet/0g_snap.tar.lz4 | tar -I lz4 -xf - -C $HOME/.evmosd

mv $HOME/.evmosd/priv_validator_state.json.backup $HOME/.evmosd/data/priv_validator_state.json
```

**Start:**

```bash
sudo systemctl daemon-reload
sudo systemctl restart evmosd
```

**Log:**

```bash
sudo journalctl -u evmosd.service -f --no-hostname -o cat
```

**Create Wallet:**

```bash
evmosd keys add wallet-name
```

**Get Wallet EVM Address:**

```bash
echo "0x$(evmosd debug addr $(evmosd keys show wallet-name -a) | grep hex | awk '{print $3}')"
```

**evm scan:**

> [https://scan-testnet.0g.ai](https://scan-testnet.0g.ai)

**EVM Private Key and Metamask:**

```bash
evmosd keys unsafe-export-eth-key wallet-name
```

**FAUCET:**
Request for a faucet to your wallet that you created before creating validator.

 [https://faucet.0g.ai/](https://faucet.0g.ai/)

**Create Validator:**

```bash
evmosd tx staking create-validator \
  --amount=10000000000000000aevmos \
  --pubkey=$(evmosd tendermint show-validator) \
  --moniker=MONIKER_NAME \
  --chain-id=zgtendermint_9000-1 \
  --commission-rate=0.05 \
  --commission-max-rate=0.10 \
  --commission-max-change-rate
