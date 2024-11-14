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

