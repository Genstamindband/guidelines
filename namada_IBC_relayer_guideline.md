***IBC relayer guideline between testnet and campfire***

# Deploy testnet-15
Refer to https://github.com/aquariusluo/guidelines/blob/main/namada_validator_guideline.md

# Deploy campfire via docker-free

## Setup environment
export CHAIN_ID="public-testnet-15.0dacadb8d663"  
export CHAIN_ID_A=$CHAIN_ID  
export CHAIN_ID_B="luminara.857cf638d323bbae2ed94"  
export BASE_DIR=$HOME/.local/share/namada  
export BASE_DIR_A=$BASE_DIR  
export BASE_DIR_B="$HOME/.local/share/campfire"  

## Create campfire service
mkdir -p $HOME/campfire/bin && cd $HOME/campfire/bin  
wget -c https://github.com/anoma/namada/releases/download/v0.28.1/namada-v0.28.1-Linux-x86_64.tar.gz  
tar -zxvf namada-v0.28.1-Linux-x86_64.tar.gz --strip-components=1 && rm namada-v0.28.1-Linux-x86_64.tar.gz  
```
sudo tee /usr/lib/systemd/user/campfired.service > /dev/null <<EOF
[Unit]
Description=Campfire Daemon Service
After=network.target
StartLimitIntervalSec=60
StartLimitBurst=3

[Service]
Environment="NAMADA_LOG=info"
Environment="CMT_LOG_LEVEL=p2p:none,pex:error"
Environment="NAMADA_CMT_STDOUT=true"
Type=simple
Restart=always
RestartSec=30
LimitNOFILE=65535
LimitSTACK=infinity
ExecStart=%h/campfire/bin/namada --chain-id=luminara.857cf638d323bbae2ed94 --base-dir=%h/.local/share/campfire node ledger run

[Install]
WantedBy=default.target
EOF
```
sudo chmod 755 /usr/lib/systemd/user/campfired.service  
sudo loginctl enable-linger namadanet  
systemctl --user daemon-reload  
systemctl --user enable campfired  

## Configure campfire
export NAMADA_NETWORK_CONFIGS_SERVER=https://testnet.luminara.icu/configs  
cd $HOME/campfire/bin  
mkdir -p ${BASE_DIR_B}/${CHAIN_ID_B}/wasm/  
cp $BASE_DIR/global-config.toml $BASE_DIR/global-config.toml.bak  
./namada client utils join-network --chain-id $CHAIN_ID_B --dont-prefetch-wasm  
mv $BASE_DIR/global-config.toml $BASE_DIR_B/ && mv $BASE_DIR/$CHAIN_ID_B $BASE_DIR_B/  
mv $BASE_DIR/global-config.toml.bak $BASE_DIR/global-config.toml  
sudo chmod -R 777 $BASE_DIR_B  

wget -c https://testnet.luminara.icu/wasm.tar.gz  
tar -zxvf wasm.tar.gz  
mv wasm/* ${BASE_DIR_B}/${CHAIN_ID_B}/wasm/ && rm wasm.tar.gz && rm -rf wasm  

### Find and/or add at least one persistent peer in config.toml:
```
curl -OL https://testnet.luminara.icu/luminara.env
grep PERSISTENT_PEERS luminara.env | cut -d'=' -f2
sed -n '/persistent_peers/s/^persistent_peers = "\(.*\)"/\1/p' $BASE_DIR_B/$CHAIN_ID_B/config.toml
```
### Modify ports in config.toml
```
sed -i 's|proxy_app = "tcp://127.0.0.1:26658"|proxy_app = "tcp://127.0.0.1:36658"|; s|laddr = "tcp://127.0.0.1:26657"|laddr = "tcp://127.0.0.1:36657"|; s|laddr = "tcp://0.0.0.0:26656"|laddr = "tcp://0.0.0.0:36656"|' $BASE_DIR_B/$CHAIN_ID_B/config.toml
```
## Add fireall rule
sudo ufw allow 36656

## Start campfire service
systemctl --user start campfired

## Check node status
journalctl --user-unit=campfired.service -f  
curl -s localhost:36657/status | jq .

# Setup IBC relayer via hermes

## Build hermes via source code
export TAG="3f28e54beb21a071250f165579524ab2b734418b"  
cd $HOME && git clone https://github.com/heliaxdev/hermes.git && cd hermes && git checkout $TAG  
cargo build --release --bin hermes  
target/release/hermes --version  
-> hermes 1.7.3+3f28e54b  
sudo cp target/release/hermes /usr/local/bin/  

## Create hermes service
```
sudo tee /usr/lib/systemd/user/hermesd.service > /dev/null <<EOF
[Unit]
Description=Hermes Daemon Service
After=network.target
StartLimitIntervalSec=60
StartLimitBurst=3

[Service]
Type=simple
Restart=always
RestartSec=30
ExecStart=/usr/local/bin/hermes --config $HOME/.hermes/config.toml start 

[Install]
WantedBy=default.target
EOF
```
sudo chmod 755 /usr/lib/systemd/user/hermesd.service  
systemctl --user daemon-reload  
systemctl --user enable hermesd  

## Create relayer accounts
Create relayer account on testnet-15:  
namadaw key gen --alias "relayer" --unsafe-dont-encrypt  
namadaw address find --alias "relayer"  
```
Found address Implicit: tnam1qr5cl4ptvkeugtpjv27gynetdkap4yxu5cgeynjs
```
Create relayer account on campfire:  
$HOME/campfire/bin/namadaw --base-dir $BASE_DIR_B key gen --alias "relayer" --unsafe-dont-encrypt  
$HOME/campfire/bin/namadaw --chain-id $CHAIN_ID_B --base-dir $BASE_DIR_B address find --alias "relayer"
```
Found address Implicit: tnam1qp7x23umthcpg556e33z9tkdv400qnga5yw2ytet
```
Check NAM address on testnet-15:
namadaw address find --alias "nam"
```
Found address Established: tnam1q9mjvqd45u7w54kee2aquugtv7vn7h3xrcrau7xy
```
Check NAM address on campfire:
$HOME/campfire/bin/namadaw --chain-id $CHAIN_ID_B --base-dir $BASE_DIR_B address find --alias "nam"
```
Found address Established: tnam1q98ulmx4gu5dws5msjy8jsw3358j7k2gsv0267tl
```

## Configure hermes
export HERMES_DIR="$HOME/.hermes"  
export HERMES_CONFIG="$HERMES_DIR/config.toml"  
export WALLET_PATH_A="$BASE_DIR_A/$CHAIN_ID_A/wallet.toml"  
export WALLET_PATH_B="$BASE_DIR_B/$CHAIN_ID_B/wallet.toml"  

mkdir $HERMES_DIR && cd $HERMES_DIR  
```
sudo tee $HERMES_CONFIG > /dev/null <<EOF
[global]
log_level = 'info'
 
[mode]

[mode.clients]
enabled = true
refresh = true
misbehaviour = true

[mode.connections]
enabled = false

[mode.channels]
enabled = false

[mode.packets]
enabled = true
clear_interval = 10
clear_on_start = false
tx_confirmation = true

[telemetry]
enabled = false
host = '127.0.0.1'
port = 3001

[[chains]]
id = 'public-testnet-15.0dacadb8d663'
type = 'Namada'
rpc_addr = 'http://127.0.0.1:26657'  # set the IP and the port of the chain
grpc_addr = 'http://127.0.0.1:9090'
event_source = { mode = 'push', url = 'ws://127.0.0.1:26657/websocket', batch_delay = '500ms' } 
account_prefix = ''
key_name = 'relayer'
store_prefix = 'ibc'
gas_price = { price = 0.0001, denom = 'tnam1q9mjvqd45u7w54kee2aquugtv7vn7h3xrcrau7xy' } # retrieve denom by namadaw address find --alias "nam"
rpc_timeout = '30s'

[[chains]]
id = 'luminara.857cf638d323bbae2ed94'  # set your chain ID
type = 'Namada'
rpc_addr = 'http://127.0.0.1:36657'  # set the IP and the port of the chain
grpc_addr = 'http://127.0.0.1:9090'  # not used for now
event_source = { mode = 'push', url = 'ws://127.0.0.1:36657/websocket', batch_delay = '500ms' } 
account_prefix = ''  # not used
key_name = 'relayer'  # The key is an account name you made
store_prefix = 'ibc'
gas_price = { price = 0.0001, denom = 'tnam1q98ulmx4gu5dws5msjy8jsw3358j7k2gsv0267tl' } # retrieve denom by $HOME/campfire/bin/namadaw --chain-id $CHAIN_ID_B --base-dir $BASE_DIR_B address find --alias "nam"
rpc_timeout = '30s'
EOF
```

## Add the relayer key to Hermes
```
hermes --config $HERMES_CONFIG keys add --chain $CHAIN_ID_A --key-file $WALLET_PATH_A --overwrite
hermes --config $HERMES_CONFIG keys add --chain $CHAIN_ID_B --key-file $WALLET_PATH_B --overwrite
ls $HERMES_DIR/keys/$CHAIN_ID_A/keyring-test/relayer.json
ls $HERMES_DIR/keys/$CHAIN_ID_B/keyring-test/relayer.json
```
## Fund faucet NAM to relayer accounts
Testnet-15: https://faucet.heliax.click/ (tnam1qr5cl4ptvkeugtpjv27gynetdkap4yxu5cgeynjs)  
Campfire: https://faucet.luminara.icu/   (tnam1qp7x23umthcpg556e33z9tkdv400qnga5yw2ytet)  

## Create IBC channel
```
hermes --config $HERMES_CONFIG \
  create channel \
  --a-chain $CHAIN_ID_A \
  --b-chain $CHAIN_ID_B \
  --a-port transfer \
  --b-port transfer \
  --new-client-connection --yes
```
Get result of channel
```
SUCCESS Channel {
    ordering: Unordered,
    a_side: ChannelSide {
        chain: BaseChainHandle {
            chain_id: ChainId {
                id: "public-testnet-15.0dacadb8d663",
                version: 0,
            },
            runtime_sender: Sender { .. },
        },
        client_id: ClientId(
            "07-tendermint-82",
        ),
        connection_id: ConnectionId(
            "connection-14",
        ),
        port_id: PortId(
            "transfer",
        ),
        channel_id: Some(
            ChannelId(
                "channel-13",
            ),
        ),
        version: None,
    },
    b_side: ChannelSide {
        chain: BaseChainHandle {
            chain_id: ChainId {
                id: "luminara.857cf638d323bbae2ed94",
                version: 0,
            },
            runtime_sender: Sender { .. },
        },
        client_id: ClientId(
            "07-tendermint-10",
        ),
        connection_id: ConnectionId(
            "connection-7",
        ),
        port_id: PortId(
            "transfer",
        ),
        channel_id: Some(
            ChannelId(
                "channel-8",
            ),
        ),
        version: None,
    },
    connection_delay: 0ns,
}
```

## Start hermes service
systemctl --user start hermesd  
journalctl --user-unit=hermesd.service -f  

## IBC-tranfer
Generate "Tom" on testnet-15:  
namadaw key gen --alias "Tom"  
Retrieve "Tom" address:  
namadaw address find --alias "Tom"  
```
Found address Implicit: tnam1qzxcxg94ynyxlxzaz888ptwcwdrkxwkxt572xq4n
```
Generate "Jonny" on campfire:  
$HOME/campfire/bin/namadaw --base-dir $BASE_DIR_B key gen --alias "Jonny"  
Retrieve "Jonny" address:  
$HOME/campfire/bin/namadaw --chain-id $CHAIN_ID_B --base-dir $BASE_DIR_B address find --alias "Jonny"  
```
Found address Implicit: tnam1qqg6xcx03zfh04k360alwlejzwha3xmucg4kcddr
```

Fund 10 btc faucet to Tom  
Fund 100 nam faucet to Tom  
Check balance before sending  
```
namadac balance --owner "Tom"
btc: 10
nam: 100

$HOME/campfire/bin/namadac --chain-id $CHAIN_ID_B --base-dir $BASE_DIR_B balance --node localhost:36657 --owner "Jonny"
n/a
```

Transfer 1 btc from Tom to Jonny
```
export CHANNEL_ID_A="channel-13"
export CHANNEL_ID_B="channel-8"
export LEDGER_ADDRESS_A="127.0.0.1:26657"
namadac --base-dir ${BASE_DIR_A} \
    ibc-transfer \
    --amount 1 \
    --source Tom \
    --receiver tnam1qqg6xcx03zfh04k360alwlejzwha3xmucg4kcddr \
    --token btc \
    --channel-id ${CHANNEL_ID_A} \
    --node ${LEDGER_ADDRESS_A}
```
Check balance received
```
namadac balance --owner "Tom"
btc: 9
nam: 99.95

$HOME/campfire/bin/namadac --chain-id $CHAIN_ID_B --base-dir $BASE_DIR_B balance --node localhost:36657 --owner "Jonny"
transfer/channel-8/btc: 1
```

## MISC
Update client:  
`hermes update client --host-chain $CHAIN_ID --client $CLIENT_ID`  
Check status of testnet-15:  
`curl -s localhost:26657/status | jq .`  
Check status of campfire:  
`curl -s localhost:36657/status | jq .`  




