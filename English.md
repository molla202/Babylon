![logo-blue-orange-dot](https://github.com/molla202/Babylon/assets/91562185/09c70e6d-b9c7-421e-9680-400c538331b1)

<h1 align="center"> Babylon Chain </h1>

 * [Community channel](https://t.me/corenodechat)<br>
 * [Community Twitter](https://twitter.com/corenodeHQ)<br>
 * [Babylon Website](https://www.babylonchain.io/)<br>
 * [Blockchain Explorer](https://babylon.explorers.guru/)<br>
 * [Discord](https://discord.gg/V7NGnHq3)<br>
 * [Twitter](https://twitter.com/babylon_chain)<br>


## System Requirements
| Components | Minimum Requirements | 
| ------------ | ------------ |
| CPU |	4 |
| RAM	| 8 GB |
| Storage	| 250 GB SSD |
### Update and lib. setup
```
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

```
### Go setup
```
cd $HOME
! [ -x "$(command -v go)" ] && {
VER="1.19.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
}
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

### Moniker name write (PORT 311 or change)

```

echo "export WALLET="wallet-name"" >> $HOME/.bash_profile
echo "export MONIKER="moniker-name"" >> $HOME/.bash_profile
echo "export BABYLON_CHAIN_ID="bbn-test-2"" >> $HOME/.bash_profile
echo "export BABYLON_PORT="311"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
```
cd || return
rm -rf babylon
git clone https://github.com/babylonchain/babylon
cd babylon || return
git checkout v0.7.2
make install
babylond version # v0.7.2

babylond config keyring-backend test
babylond config chain-id bbn-test-2
babylond init "$MONIKER" --chain-id bbn-test-2

curl -Ls https://raw.githubusercontent.com/Core-Node-Team/Testnet-TR/main/Babylon/genesis.json > $HOME/.babylond/config/genesis.json

curl -Ls https://raw.githubusercontent.com/Core-Node-Team/Testnet-TR/main/Babylon/addrbook.json > $HOME/.babylond/config/addrbook.json

SEEDS="8da45f9ff83b4f8dd45bbcb4f850999637fbfe3b@seed0.testnet.babylonchain.io:26656,4b1f8a774220ba1073a4e9f4881de218b8a49c99@seed1.testnet.babylonchain.io:26656"
PEERS=""
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.babylond/config/config.toml

sed -i 's|^pruning *=.*|pruning = "custom"|g' $HOME/.babylond/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $HOME/.babylond/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $HOME/.babylond/config/app.toml
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 0|g' $HOME/.babylond/config/app.toml

sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.00001ubbn"|g' $HOME/.babylond/config/app.toml
sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.babylond/config/config.toml
sed -i 's|^network *=.*|network = "mainnet"|g' $HOME/.babylond/config/app.toml

sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${BABYLON_PORT}317\"%;
s%^address = \":8080\"%address = \":${BABYLON_PORT}080\"%;
s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${BABYLON_PORT}090\"%; 
s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${BABYLON_PORT}091\"%; 
s%^address = \"0.0.0.0:8545\"%address = \"0.0.0.0:${BABYLON_PORT}545\"%; 
s%^ws-address = \"0.0.0.0:8546\"%ws-address = \"0.0.0.0:${BABYLON_PORT}546\"%" $HOME/.babylond/config/app.toml

sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${BABYLON_PORT}658\"%; 
s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${BABYLON_PORT}657\"%; 
s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${BABYLON_PORT}060\"%;
s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${BABYLON_PORT}656\"%;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${BABYLON_PORT}656\"%;
s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${BABYLON_PORT}660\"%" $HOME/.babylond/config/config.toml


sudo tee /etc/systemd/system/babylond.service > /dev/null << EOF
[Unit]
Description=Babylon Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which babylond) start --x-crisis-skip-assert-invariants
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF


sudo systemctl daemon-reload
sudo systemctl enable babylond
sudo systemctl start babylond

sudo journalctl -u babylond -f --no-hostname -o cat
```

### wallet
```
babylond keys add $WALLET
```
### wallet import
```
babylond keys add $WALLET --recover
```

## ⚡⚡⚡ BLS key create ⚡⚡⚡
```
babylond create-bls-key $(babylond keys show $WALLET -a)
```
### And reset node 
```
sudo systemctl restart babylond
```
### Senkron false oke
```
babylond status 2>&1 | jq .SyncInfo
```
### Validator create ( moniker and wallet name change)
```
babylond tx checkpointing create-validator \
--amount=10000000ubbn \
--pubkey=$(babylond tendermint show-validator) \
--moniker="moniker-name" \
--identity= \
--details="Mustafa Kemal ATATÜRK" \
--chain-id=bbn-test-2 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=$WALLET \
--gas-prices=0.1ubbn \
--gas-adjustment=1.5 \
--gas=auto \
-y
```



