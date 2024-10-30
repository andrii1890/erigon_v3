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
     ```sudo nano /etc/systemd/system/erigon.service```
     ```
     [Unit]
     Description=Ethereum Node Service
     After=network-online.target

     [Service]
     User=root
     ExecStart=erigon --config /root/ethereum/eth1/config/config.yaml
     Restart=on-failure
     RestartSec=10
     LimitNOFILE=65535

     StandardOutput=append:/root/ethereum/eth1/logs/erigon.log
     StandardError=append:/root/ethereum/eth1/logs/erigon.log

    [Install]
    WantedBy=multi-user.target
    ```
    ```
    sudo systemctl enable erigon.service
    ```
- **Create config file**
   ```
   nano /$HOME/ethereum/eth1/config/config.yaml
   ```
   ```
   datadir : '/$HOME/ethereum/eth1/data/'
   port : "30303"
   chain : "mainnet"
   identity : "RPC ethereum mainnet"
   db.size.limit : "4TB"
   nat : "extip:Your Public IP"
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
    echo 'alias erigon_log="tail -f /$HOME/ethereum/eth1/logs/erigon.log"' >> $HOME/.profile
    source $HOME/.profile
    ```
    now you can simply find logs: erigon_log

***HTTP*** request will be available on: ***http://<YOUR_IP>:8545***
  
***WS*** request will be available on: ***ws://<YOUR_IP>:8546***
  
**Beacon Client*** request will be available on: ***http://<YOUR_IP>:5555***
