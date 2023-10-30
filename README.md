# SanchoNet-SPO
## Guide for SanchoNet stake pool operators (SPOs)
### Ubuntu 22.04 LTS (x86)

#### Prerequisites

```
sudo apt-get update -y && sudo apt-get upgrade -y
sudo apt-get install automake build-essential pkg-config libffi-dev libgmp-dev libssl-dev libtinfo-dev libsystemd-dev zlib1g-dev make g++ tmux git jq wget libncursesw5 libtool autoconf -y
```


##### Install cabal

```
curl --proto '=https' --tlsv1.2 -sSf https://get-ghcup.haskell.org | sh
```
Options: P, N, N, ENTER

```
ghcup install ghc 8.10.7
ghcup set ghc 8.10.7
```
```
ghcup install cabal 3.8.1.0
ghcup set cabal 3.8.1.0
```
```
ghc --version
cabal --version
```
```
mkdir -p $HOME/sancho-src
cd $HOME/sancho-src
```
#### Libsodium

```
git clone https://github.com/input-output-hk/libsodium
cd libsodium
git checkout dbb48cc
./autogen.sh
./configure
make
sudo make install
```
```
cd
nano .bashrc
```
add these two lines to your .bashrc

```
export LD_LIBRARY_PATH="/usr/local/lib:$LD_LIBRARY_PATH"
export PKG_CONFIG_PATH="/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH"
```
source it, to activate those paths on current terminal

```
source .bashrc
```

#### BLST

```
cd $HOME/sancho-src
git clone https://github.com/supranational/blst
cd blst
git checkout v0.3.10
./build.sh
cat > libblst.pc << EOF
prefix=/usr/local
exec_prefix=\${prefix}
libdir=\${exec_prefix}/lib
includedir=\${prefix}/include

Name: libblst
Description: Multilingual BLS12-381 signature library
URL: https://github.com/supranational/blst
Version: 0.3.10
Cflags: -I\${includedir}
Libs: -L\${libdir} -lblst
EOF
sudo cp libblst.pc /usr/local/lib/pkgconfig/
sudo cp bindings/blst_aux.h bindings/blst.h bindings/blst.hpp  /usr/local/include/
sudo cp libblst.a /usr/local/lib
sudo chmod u=rw,go=r /usr/local/{lib/{libblst.a,pkgconfig/libblst.pc},include/{blst.{h,hpp},blst_aux.h}}
```

#### Build cardano-node and cardano-cli
```
cd ..
git clone https://github.com/input-output-hk/cardano-node.git
cd cardano-node
git fetch --all --recurse-submodules --tags
```

**version 8.5.0 for SanchoNet**

```
git checkout f1ce770834bf7150ca29cb647065c9e62d39be1a
```

```
cabal configure --with-compiler=ghc-8.10.7
```

```
cabal update
cabal build cardano-node cardano-cli
```

```
mkdir -p $HOME/.local/bin
cp -p "$(./scripts/bin-path.sh cardano-node" $HOME/.local/bin/
cp -p "$(./scripts/bin-path.sh cardano-cli" $HOME/.local/bin/
```

```
cardano-node --version
cardano-cli --version
```

#### cardano-node as a Linux service, sancho-node.service

```
sudo nano /etc/systemd/system/sancho-node.service
```


```
# The Cardano node service (part of systemd)
# file: /etc/systemd/system/sancho-node.service

 [Unit]
Description       = Cardano Node Service
Wants             = network-online.target
After             = network-online.target

[Service]
User=sancho
Type=simple
WorkingDirectory=/home/sancho/sanchonet/
ExecStart=/home/sancho/sanchonet/startnode.sh
KillSignal=SIGINT
RestartKillSignal=SIGINT
TimeoutStopSec=300
LimitNOFILE=32768
Restart=always
RestartSec=5
SyslogIdentifier=sancho-node

 [Install]
WantedBy          = multi-user.target
```

#### Start sancho-node (startnode.sh)

```
cd
mkdir sanchonet
cd sanchonet
nano startnode.sh
```

Copy the start script:

```
#!/bin/bash

# Configuration variables
TOPOLOGY_FILE="/home/sancho/sanchonet/topology.json"
CONFIG_FILE="/home/sancho/sanchonet/config.json"
DATABASE_PATH="/home/sancho/sanchonet/db"
SOCKET_PATH="/home/sancho/sanchonet/db/node.socket"
HOST_ADDR="0.0.0.0"
PORT="3000"

/home/sancho/.local/bin/cardano-node run \
    --topology "${TOPOLOGY_FILE}" \
    --database-path "${DATABASE_PATH}" \
    --socket-path "${SOCKET_PATH}" \
    --host-addr "${HOST_ADDR}" \
    --port "${PORT}" \
    --config "${CONFIG_FILE}"
```
Give execution permission to the start script
```
chmod +x startnode.sh 
```

Enable the service:

```
sudo systemctl daemon-reload
sudo systemctl enable sancho-node.service
```

Check status of the service:

```
sudo systemctl status sancho-node.service
```



Other commands
```
sudo systemctl stop sancho-node.service
```

```
sudo systemctl start sancho-node.service
```

```
journalctl --unit=sancho-node --follow
```
