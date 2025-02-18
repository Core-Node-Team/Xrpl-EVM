```
sudo apt install liblz4-tool
systemctl stop xrpd
cp $HOME/.xrpd/data/priv_validator_state.json $HOME/.xrpd/priv_validator_state.json.backup
cp $HOME/.xrpd/config/priv_validator_key.json $HOME/.xrpd/priv_validator_key.json.backup
xrpdd tendermint unsafe-reset-all --home $HOME/.xrpd --keep-addr-book
curl -L http://37.120.189.81/xrpl/xrpl_snap.lz4 | tar -I lz4 -xf - -C $HOME/.xrpd
mv $HOME/.xrpd/priv_validator_state.json.backup $HOME/.xrpd/data/priv_validator_state.json
sudo systemctl daemon-reload
sudo systemctl restart xrpdd
sudo journalctl -u xrpdd.service -f --no-hostname -o cat
```
