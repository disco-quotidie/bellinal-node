# Bellinal-node
Bellscoin full node and bells-ord deployment

## Bellscoin full node
Prepare for a Ubuntu server and login as root.

### Clone the bells-core repository
```
wget https://github.com/Nintondo/bellscoinV3/releases/download/v3.0.0-rc2/bells-3.0.0-rc2-x86_64-linux-gnu.tar.gz
```

### Extract the compressed file and rename to bellscoin
```
tar -xvzf bells-3.0.0-rc2-x86_64-linux-gnu.tar.gz
mv bells-3.0.0-rc2 bellscoin
rm bells-3.0.0-rc2-x86_64-linux-gnu.tar.gz
```

### Install dependencies
```
apt-get install build-essential libtool autotools-dev automake pkg-config bsdmainutils python3
apt install -y libevent-dev libboost-dev libboost-system-dev libboost-filesystem-dev libboost-test-dev libboost-thread-dev libsqlite3-dev
```

### Install Bellscoin binary to system path
```
sudo install -m 0755 -o root -g root -t /usr/local/bin bellscoin/bin/*
```

### Make Bellscoin data directory
```
mkdir -p ~/.bells
```

### Write configuration file for bells-core RPC
```
cat >~/.bells/bells.conf <<EOF
server=1
rpcuser=your-rpc-username
rpcpassword=your-rpc-password
rpcallowip=127.0.0.1
rpcbind=0.0.0.0
rpcport=8332
listen=1
daemon=1
txindex=1
EOF
```

### Make your bellscoin core system daemon
```
cat >/etc/systemd/system/bellsd.service <<EOF
[Unit]
Description=Bellscoin daemon
After=network.target

[Service]
ExecStart=/usr/local/bin/bellsd -daemon -conf=/root/.bells/bells.conf -pid=/root/.bells/bellsd.pid
User=root
Group=root
Type=forking
PIDFile=/root/.bells/bellsd.pid
Restart=on-failure
RestartSec=5
KillMode=process

[Install]
WantedBy=multi-user.target
EOF
```

### Start your daemon
```
systemctl daemon-reload
systemctl enable bellsd.service
systemctl start bellsd.service
```

Your IBD (Initial Block Download) gets started. You can track current status with this command.
```
bells-cli getblockchaininfo
```


## Bellscoin Ord
### Install dependencies
```
apt-get install pkg-config libssl-dev build-essential
```

### Install Rust
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
Relogin to user and verify Rust is installed
```
rustc --version
```

### Clone Bellscoin Ord repository
```
git clone https://github.com/Nintondo/ord.git
```
Replace RPC port on Ord repository (Deprecation issue)
```
vim ord/src/chains.rs
```
Replace 19918 to 8332 on default_rpc_port function

### Build Rust Binary
```
cd ord
cargo build --release
cd target/release
```

### Install Latest NodeJS and PM2
```
sudo apt install -y ca-certificates curl gnupg
sudo mkdir -p /etc/apt/keyrings
sudo curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg
NODE_MAJOR=20
echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list
sudo apt update -y 
sudo apt install -y nodejs
npm i -g pm2@latest
```

### Run Ord as service with PM2
```
pm2 start "./ord --bitcoin-rpc-username your-rpc-username --bitcoin-rpc-password your-rpc-password --index-addresses server" --name ord
```

### Verify Ord service running on port 3333
http://ip:3333
