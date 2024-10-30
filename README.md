# erigon
erigon archive node
## First step
create service file
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
