**This is a guideline for Namada pre-genesis validator node**  
    
_Server requirement is 4vCPU, 16GB, 1TB SSD_   
_Current Vesion is testnet-15, CHAIN_ID="public-testnet-15.0dacadb8d663"_  
_root user 'namadanet' has been created in advanced_  

# Install Pre-requisites software
```
sudo apt update -y  
sudo apt-get install -y make git-core libssl-dev pkg-config libclang-12-dev build-essential protobuf-compiler libudev-dev  
sudo curl https://sh.rustup.rs -sSf | sh -s -- -y  
. $HOME/.cargo/env  
```
# Setup firewall
```
sudo ufw allow 22  
sudo ufw allow 26656
sudo ufw allow 26657  (RPC)
sudo ufw allow 26660  (for namada metic)
sudo ufw allow 9100   (for node-exporter)
sudo ufw enable
```
# Config enviornment
vi ~/.bash_profile
```
export PATH=$HOME/.cargo/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/usr/local/go/bin:$HOME/go/bin  
export BASE_DIR=$HOME/.local/share/namada  
export NAMADA_TAG=v0.28.2    
export TM_HASH=v0.1.4-abciplus  
export CHAIN_ID="public-testnet-15.0dacadb8d663"  
export PUBLIC_IP="<PUBLIC IP>"  
export IP_PORT="<PUBLIC IP>:26656"  
export VALIDATOR_ALIAS="<VALIDATOR MONIKER>"  
. "$HOME/.cargo/env"  
```
source ~/.bash_profile

# Build from source code
Build namada and copy to /usr/local/bin
```
cd $HOME && git clone https://github.com/anoma/namada && cd namada && git checkout $NAMADA_TAG
make build-release

cd $HOME && sudo cp "$HOME/namada/target/release/namada" /usr/local/bin/namada && sudo cp "$HOME/namada/target/release/namadac" /usr/local/bin/namadac && sudo cp "$HOME/namada/target/release/namadan" /usr/local/bin/namadan && sudo cp "$HOME/namada/target/release/namadaw" /usr/local/bin/namadaw && sudo cp "$HOME/namada/target/release/namadar" /usr/local/bin/namadar
```
Install consensus cometbft
```
wget https://github.com/cometbft/cometbft/releases/download/v0.37.2/cometbft_0.37.2_linux_amd64.tar.gz
tar -xvf cometbft_0.37.2_linux_amd64.tar.gz

sudo cp cometbft /usr/local/bin/
```
Verify version
```
cometbft version
0.37.2
namada --version
Namada v0.28.2
```

# Copy pre-genesis folder to $BASE_DIR
```
mkdir -p $BASE_DIR
sudo cp -r ~/pre-genesis $BASE_DIR
```

# Join network
```
cd $HOME && namadac utils join-network --chain-id $CHAIN_ID --genesis-validator $VALIDATOR_ALIAS
```
# Modify $BASE_DIR/$CHAIN_ID/config.toml
```
[ledger.cometbft.rpc]
laddr = "tcp://0.0.0.0:26657"
[ledger.cometbft.instrumentation]
prometheus = true
prometheus_listen_addr = ":26660"
[ledger.cometbft]
moniker = "<your-moniker-name>"
```
# Create and launch service
sudo vi /etc/systemd/system/namadad.service
```
[Unit]
Description=namada
After=network-online.target

[Service]
User=namadanet
WorkingDirectory=/home/namadanet/.local/share/namada
Environment="NAMADA_LOG=info"
Environment="CMT_LOG_LEVEL=p2p:none,pex:error"
Environment="NAMADA_CMT_STDOUT=true"
ExecStart=/usr/local/bin/namada --base-dir=/home/namadanet/.local/share/namada node ledger run  
StandardOutput=syslog
StandardError=syslog
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```
sudo chmod 755 /etc/systemd/system/namadad.service  
sudo systemctl daemon-reload  
sudo systemctl enable namadad  
sudo systemctl start namadad && sudo journalctl -u namadad -n 1000 -f  

# Check status
```
curl -s localhost:26657/status | jq .  
namadac find-validator --tm-address=$(curl -s localhost:26657/status | jq -r .result.validator_info.address)  
namadac validator-state --validator <validator address>
```
# Service operations
```
sudo service namadad start  
sudo service namadad status  
sudo service namadad stop   
sudo service namadad restart  
sudo journalctl -u namadad -n 1000 -f | grep "height"  
```
# MISC
Derive pre-genesis key from mnemonic words  
`namadaw --pre-genesis key derive --alias <your_moniker>`  

Check bonded-stake  
`namadac bonded-stake | grep <account_address>`  

Check balances/bonds  
```
namadac balance --owner $VALIDATOR_ALIAS --token NAM
namadac bonds --owner $VALIDATOR_ALIAS
```

Check key/address list
```
namadaw key list
namadaw address list
```
Backup/restore
```
config.toml, wallet.toml, priv_validator_key.json, node_key.json
```
Snapshot
```
wget https://files.cryptosj.net/files/namadatestnet/namadatestnet.tar.gz
```
# Monitor tools
TG bot: https://t.me/NamTool_Bot from https://namtool.denodes.xyz/   
Prometheus: https://github.com/aquariusluo/prometheus-grafana-exporter
