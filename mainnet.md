![image](https://github.com/user-attachments/assets/fea8d094-7856-43cd-9c76-b02be353d055)

 
 * [Topluluk kanalımız](https://t.me/corenodechat)<br>
 * [Topluluk Twitter](https://twitter.com/corenodeHQ)<br>


### Explorer And RPC

https://explorer.corenodehq.xyz/xrplevm-mainnet/staking

https://xrpl-mainnet-rpc.corenodehq.xyz

https://xrpl-mainnet-api.corenodehq.xyz

https://xrpl-mainnet-evm.corenodehq.xyz

### Gereklilikler
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```
### GO
```
cd $HOME
VER="1.23.4"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```
### Varyasyonlar
```
echo "export XRPL_WALLET="mictowallet"" >> $HOME/.bash_profile
echo "export XRPL_PORT="48"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
```
mkdir -p $HOME/.exrpd/cosmovisor/upgrades/v8.0.0/bin
rm -rf xrpl-mainnet
git clone https://github.com/xrplevm/node.git xrpl-mainnet
cd xrpl-mainnet
git checkout v8.0.0
make build
mv $HOME/xrpl-mainnet/bin/exrpd $HOME/.exrpd/cosmovisor/upgrades/v8.0.0/bin/
```
```
sudo ln -sfn $HOME/.exrpd/cosmovisor/upgrades/v8.0.0 $HOME/.exrpd/cosmovisor/current
sudo ln -sfn $HOME/.exrpd/cosmovisor/current/bin/exrpd /usr/local/bin/exrpd
```
```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.6.0
cd
```
### Servis
```
sudo tee /etc/systemd/system/exrpd.service > /dev/null <<EOF
[Unit]
Description=xrpl node service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start --home $HOME/.exrpd
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.exrpd"
Environment="DAEMON_NAME=exrpd"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.exrpd/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload
sudo systemctl enable exrpd
```
### Init
```
exrpd init "Moniker-Name" --chain-id exrp_1440000-1
```
```
sed -i -e '/^chain-id = /c\chain-id = "exrp_1440000-1"' $HOME/.exrpd/config/client.toml
sed -i -e "s|^node *=.*|node = \"tcp://localhost:${XRPL_PORT}657\"|" $HOME/.exrpd/config/client.toml
sed -i -e '/^keyring-backend = /c\keyring-backend = "test"' $HOME/.exrpd/config/client.toml
```
### Genesis And Addrbook
```
curl https://raw.githubusercontent.com/Core-Node-Team/Xrpl-EVM/refs/heads/main/mainnet-genesis.json -o ~/.exrpd/config/genesis.json
```
```
curl https://raw.githubusercontent.com/Core-Node-Team/Xrpl-EVM/refs/heads/main/mainnet-addrbook.json -o ~/.exrpd/config/addrbook.json
```
### Port ayarları
```
sed -i.bak -e "s%:26658%:${XRPL_PORT}658%g;
s%:26657%:${XRPL_PORT}657%g;
s%:6060%:${XRPL_PORT}060%g;
s%:26656%:${XRPL_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${XRPL_PORT}656\"%;
s%:26660%:${XRPL_PORT}660%g" $HOME/.exrpd/config/config.toml
```
```
sed -i.bak -e "s%:1317%:${XRPL_PORT}317%g;
s%:8080%:${XRPL_PORT}080%g;
s%:9090%:${XRPL_PORT}090%g;
s%:9091%:${XRPL_PORT}091%g;
s%:8545%:${XRPL_PORT}545%g;
s%:8546%:${XRPL_PORT}546%g;
s%:6065%:${XRPL_PORT}065%g" $HOME/.exrpd/config/app.toml
```
### Peers and Seeds
```
SEEDS=""
PEERS="2dc8d776176a275bc64421e31d9b1ed6af228fa5@seed-0.mainnet.xrplevm.org:26656,8f8183786ac761e90998d6ab582ab5c0b1cb8133@8.52.196.180:53656,3620f7e655bf094114e5301d9733323b0c283107@195.20.18.175:26656,5227628bdceb67b8d57f40a8b406568882b8fb8d@168.119.143.51:10656,e1279ea70948c34cd676ca9d69bf28a452cfbe03@54.39.128.229:26636"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.exrpd/config/config.toml
```
### Pruning
```
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.exrpd/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.exrpd/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"10\"/" $HOME/.exrpd/config/app.toml
```
### Indexer & Other
```
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.exrpd/config/config.toml
```
### Gas Settings
```
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0axrp"|g' $HOME/.exrpd/config/app.toml
```
### Prometheus
```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.exrpd/config/config.toml
```
## Starter Snap
```
curl -L https://snapshot.corenodehq.xyz/xrpl/xrpl_mainnet_snap.lz4 | tar -Ilz4 -xf - -C $HOME/.xrpd
```
### Başlatalım
```
sudo systemctl restart exrpd
sudo journalctl -fu exrpd -o cat
```
### Log
```
sudo journalctl -fu exrpd -o cat
```
### Cüzdan oluşturma

```
exrpd keys add $XRPL_WALLET
```
### Cüzdan import
```
exrpd keys add $XRPL_WALLET --recover
```
### Cüzdan private key al
```
exrpd keys unsafe-export-eth-key $XRPL_WALLET
```
### Validator oluşturma
NOT: gerekli yerleri değiştirin
```
rm -rf /root/validator.json
```
```
cat > ./validator.json << EOF
{
	"pubkey": $(exrpd tendermint show-validator),
	"amount": "1000000apoa",
	"moniker": "Moniker-Name",
	"identity": "optional identity signature (ex. UPort or Keybase)",
	"website": "validator's (optional) website",
	"security": "validator's (optional) security contact email",
	"details": "validator's (optional) details",
	"commission-rate": "0.1",
	"commission-max-rate": "0.2",
	"commission-max-change-rate": "0.05",
	"min-self-delegation": "1"
}
EOF
```
```
exrpd tx staking create-validator ./validator.json --chain-id=exrp_1440000-1 --gas="auto" --gas-prices="240000000000900000axrp" --from=$XRPL_WALLET -y
```
### Kendine delege
```
exrpd tx staking delegate $(exrpd keys show $XRPL_WALLET --bech val -a) 1000000apoa --from $XRPL_WALLET --chain-id exrp_1440000-1 --gas="300000" --gas-prices="240000000000900000axrp" -y 
```
### Validator edit
```
exrpd tx staking edit-validator \
--chain-id exrp_1440000-1 \
--commission-rate 0.05 \
--new-moniker "validator-name" \
--identity "" \
--details "" \
--website "" \
--security-contact "" \
--from $XRPL_WALLET \
--node http://localhost:${XRPL_PORT}657 \
--gas="300000" --gas-prices="240000000000900000axrp" \
-y
```
### Silme
```
cd $HOME
sudo systemctl stop exrpd
sudo systemctl disable exrpd
sudo rm -rf /etc/systemd/system/exrpd.service
sudo systemctl daemon-reload
sudo rm -f /usr/local/bin/exrpd
sudo rm -f $(which exrpd)
sudo rm -rf $HOME/.exrpd
sed -i "/XRPL_PORT_/d" $HOME/.bash_profile
```
