# Artela

Manual Installation

Server Preparation

    apt update && apt upgrade -y
    apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y

Install GO

    ver="1.20.3"
    wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
    sudo rm -rf /usr/local/go
    sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
    rm "go$ver.linux-amd64.tar.gz"
    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
    source $HOME/.bash_profile
    go version

Node Installation

Create node name

    NODE_MONIKER="Yourd_name_node"

next: 

    cd $HOME
    rm -rf artela
    git clone https://github.com/artela-network/artela
    cd artela
    git checkout v0.4.7-rc4
    make install

.

     artelad config chain-id artela_11822-1
     artelad init "$NODE_MONIKER" --chain-id artela_11822-1    

 .

     curl -s https://t-ss.nodeist.net/artela/genesis.json > $HOME/.artelad/config/genesis.json
     curl -s https://t-ss.nodeist.net/artela/addrbook.json > $HOME/.artelad/config/addrbook.json

 next

     SEEDS=""
     PEERS="b23bc610c374fd071c20ce4a2349bf91b8fbd7db@65.108.72.233:11656"
     sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.artelad/config/config.toml

 next

     sed -i 's|^pruning *=.*|pruning = "custom"|g' $HOME/.artelad/config/app.toml
     sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $HOME/.artelad/config/app.toml
     sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $HOME/.artelad/config/app.toml
     sed -i 's|^snapshot-interval *=.*|snapshot-interval = 0|g' $HOME/.artelad/config/app.toml

 next

     sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.025art"|g' $HOME/.artelad/config/app.toml
     sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.artelad/config/config.toml

 next

     sudo tee /etc/systemd/system/artelad.service > /dev/null << EOF
     [Unit]
     Description=artela node
     After=network-online.target
     [Service]
     User=$USER
     ExecStart=$(which artelad) start
     Restart=on-failure
     RestartSec=10
     LimitNOFILE=10000
     [Install]
     WantedBy=multi-user.target
     EOF

next

        artelad tendermint unsafe-reset-all --home $HOME/.artelad --keep-addr-book

 next

        curl -L https://t-ss.nodeist.net/artela/snapshot_latest.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.artelad --strip-components 2

next

        sudo systemctl daemon-reload
        sudo systemctl enable artelad
        sudo systemctl start artelad

Check Logs

        sudo journalctl -fu artelad -o cat

Check Syn

        artelad status 2>&1 | jq .SyncInfo.catching_up

            
