# Erigon Archive Node
  **Erigon by default is "all in one binary" solution Consensus Layer + Execution Layer**
  
  - Execution Layer - Erigon
  - Consensus Layer - Caplin
  
    Caplin is a full-fledged validating Consensus Client like Prysm, Lighthouse, Teku, Nimbus and Lodestar. Its goal is:
     - provide better stability
     - validation of the chain
     - stay in sync
     - keep the execution of blocks on chain tip
     - serve the Beacon API using a fast and compact data model alongside low CPU and memory usage.

    Caplin's Usage
    
       Caplin is be enabled by default. to disable it and enable the Engine API, use the --externalcl flag. from that point on, an external Consensus Layer will not be need anymore.

       Caplin also has an archivial mode for historical states and blocks. it can be enabled through the --caplin.archive flag. In order to enable the caplin's Beacon API, the flag --beacon.api=<namespaces> must be added. e.g: -- 
       beacon.api=beacon,builder,config,debug,node,validator,lighthouse will enable all endpoints.
    
    **NOTE: Caplin is not staking-ready so aggregation endpoints are still to be implemented. Additionally enabling the Beacon API will lead to a 6 GB higher RAM usage.



    
  **Important defaults: Erigon is an Archive Node by default: use --prune.mode if need make it smaller (not allowed to change after first start)!!!**
  - Archive Node is default. 
  - Full Node: --prune.mode=full
  - Minimal Node (EIP-4444): --prune.mode=minimal
  
    ArchiveNode Ethereum Mainnet: 3.1TB (October 2024) and FullNode: 1.5TB (October 2024)
  
  
## Hardware Requirements
- At least 4 cores, 8 threads
- 32-GB RAM or more
- 3.89Tb - NVMe SSD
- SSD or NVMe. Do not recommend HDD - on HDD Erigon will always stay N blocks behind chain tip, but not fall behind. Bear in mind that SSD performance deteriorates when close to capacity
- Golang >= 1.22; GCC 10+ or Clang; On Linux: kernel > v4. 64-bit architecture
- Internet Connection: A stable, high-speed internet connection and uninterrupted power supply is crucial!

## How much RAM do I need
- Baseline (ext4 SSD): 16Gb RAM sync takes 6 days, 32Gb - 5 days, 64Gb - 4 days
- +1 day on "zfs compression=off". +2 days on "zfs compression=on" (2x compression ratio). +3 days on btrfs.
- -1 day on NVMe

## Default Ports and Firewalls##
  Erigon ports
  - engine	9090	TCP	gRPC Server	Private
  - engine	42069	TCP & UDP	Snap sync (Bittorrent)	Public
  - engine	8551	TCP	Engine API (JWT auth)	Private
  - sentry	30303	TCP & UDP	eth/68 peering	Public
  - sentry	30304	TCP & UDP	eth/67 peering	Public
  - sentry	9091	TCP	incoming gRPC Connections	Private
  - rpcdaemon	8545	TCP	HTTP & WebSockets & GraphQL	Private
    
 Caplin ports
  - sentinel	4000	UDP	Peering	Public
  - sentinel	4001	TCP	Peering	Public
  - rest	5555	tcp	rest	Public
  
## First step
- **Update packages**
    ```
    sudo apt update && sudo apt upgrade -y
    ```
- **Install dependencies**
     ```
     sudo apt install curl build-essential git wget jq make gcc nano htop lz4  -y
     ```
- **Install GO**
    ```
    sudo rm -rf /usr/local/go
    curl -Ls https://go.dev/dl/go1.22.8.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
    eval $(echo 'export PATH=$PATH:/usr/local/go/bin' | sudo tee /etc/profile.d/golang.sh)
    eval $(echo 'export PATH=$PATH:$HOME/go/bin' | tee -a $HOME/.profile)
    ```
- **Create folders**
     ```
     cd $HOME && mkdir -p ethereum/eth1/data ethereum/eth1/config ethereum/eth1/logs
     ```
- **Create service file**
     ```
     sudo tee /etc/systemd/system/erigon.service > /dev/null << EOF
     [Unit]
     Description=Ethereum Node Service
     After=network-online.target

     [Service]
     User=$USER
     ExecStart=$(which erigon) --config $HOME/ethereum/eth1/config/config.yaml
     Restart=on-failure
     RestartSec=10
     LimitNOFILE=65535

     StandardOutput=append:$HOME/ethereum/eth1/logs/erigon.log
     StandardError=append:$HOME/ethereum/eth1/logs/erigon.log

    [Install]
    WantedBy=multi-user.target
    EOF
    sudo systemctl daemon-reload
    sudo systemctl enable erigon.service
    ```
- **Create config file**
   ```
   nano $HOME/ethereum/eth1/config/config.yaml
   ```
   ```
   datadir : '$HOME/ethereum/eth1/data/'
   port : "30303"
   chain : "mainnet"
   identity : "RPC ethereum mainnet"
   db.size.limit : "4TB"
   nat : "extip:Your Public IP"
   #nat : "any"
   http : "true"
   http.addr : "0.0.0.0"
   http.port : "8545"
   http.vhosts : "*"
   http.api : ["eth","engine","debug","net","trace","web3","erigon","txpool","admin","ots"]
   ws : "true"
   ws.port : "8546"
   torrent.port : "42069"
   torrent.download.rate : "900mb"
   metrics : "true"
   metrics.addr : "127.0.0.1"
   metrics.port : "6060"
   diagnostics.endpoint.addr : "127.0.0.1"
   diagnostics.endpoint.port : "6060"
   beacon.api.addr : "0.0.0.0"
   beacon.api.port : "5555"
   beacon.api.cors.allow-methods : ["GET","POST","PUT","OPTIONS"]
   beacon.api.cors.allow-origins : "*"
   beacon.api : ["beacon","builder","config","debug","node","lighthouse"]
   private.api.addr : "localhost:9090"
   internalcl : "true"
   netrestrict : ["10.0.0.0/8","172.16.0.0/12","100.64.0.0/10","198.18.0.0/15","169.254.0.0/16","172.16.0.0/12","192.0.2.0/24","192.88.99.0/24","192.168.0.0/16","198.18.0.0/15","198.51.100.0/24","203.0.113.0/24","224.0.0.0/4","240.0.0.0/4","192.0.0.0/24","0.0.0.0/8","255.255.255.255/32"]
   #txpool.globalbasefeeslots : "100000"
   #txpool.globalqueue : "100000"
   #txpool.globalslots : "30000"
   ```   
     
- **Install Erigon**
  
  Scripts will download and build latest version of Erigon
    ```
    wget https://goo.su/M7mbZ -O erigon_install.sh && chmod +x erigon_install.sh && ./erigon_install.sh
    ```
- **Enable Erigon Service**
   ```
   sudo systemctl start erigon.service && sudo systemctl status erigon.service
   ```
   
## Third step
- **Add alias for erigon logs**
    ```
    echo "#Erigon Logs" >> $HOME/.profile
    echo 'alias erigon_log="tail -f $HOME/ethereum/eth1/logs/erigon.log"' >> $HOME/.profile
    source $HOME/.profile
    ```
    now you can simply find logs: erigon_log

***HTTP*** request will be available on: ***http://<YOUR_IP>:8545***
  
***WS*** request will be available on: ***ws://<YOUR_IP>:8546***
  
**Beacon Client*** request will be available on: ***http://<YOUR_IP>:5555***
