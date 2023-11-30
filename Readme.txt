docker-compose installation
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/bin/docker-compose

sudo chmod +x /usr/bin/docker-compose

Downloading the Fabric images and binaries
--------------------------------------------
curl -sSL https://bit.ly/2ysbOFE | bash -s -- 2.2.4 1.4.9

sudo docker pull couchdb:3.1.1

Copy bin folder from fabric sample and paste inside bin folder we are mapping : /home/HLF_Node_Setup/bin/
Generating the crypto materials
--------------------------------
/home/HLF_Node_Setup/bin/cryptogen generate --config=/home/HLF_Node_Setup/config/crypto-config-orderer.yaml --output="/home/HLF_Node_Setup/cryptos/"
/home/HLF_Node_Setup/bin/cryptogen generate --config=/home/HLF_Node_Setup/config/crypto-config-School.yaml --output="/home/HLF_Node_Setup/cryptos/"
/home/HLF_Node_Setup/bin/cryptogen generate --config=/home/HLF_Node_Setup/config/crypto-config-College.yaml --output="/home/HLF_Node_Setup/cryptos/"
/home/HLF_Node_Setup/bin/cryptogen generate --config=/home/HLF_Node_Setup/config/crypto-config-Uni.yaml --output="/home/HLF_Node_Setup/cryptos/"
** School Folder inside cryptos folder created


Generating the gensis block
------------------------------
** create configtx.yaml inside cryptos folder
export FABRIC_CFG_PATH=/home/HLF_Node_Setup/cryptos
/home/HLF_Node_Setup/bin/configtxgen -profile ApplicationGenesisNwk -channelID testchainid -outputBlock /home/HLF_Node_Setup/genesis.block
** genesis.block file created inside main folder

Create the Docker compose file
------------------------------
create docker-compose start inside this folder  /home/HLF_Node_Setup/cryptos/peerOrganizations/School/artifacts rite
run 'sudo docker compose up -d' inside artifacts folder ( peer0.School, couchdb.School, certauth.School 3 containers started)
port and name can be change for other docker-compose file

create docker-compose file inside artifacts folder and run sudo docker compose up -d and sudo docker ps to check 3 containers for 3 nodes

Copy genesis.block file inside orderer.orederer/genesis folder
---------------------------------------------------------------
copy genesis.block inside '/home/HLF_Node_Setup/cryptos/ordererOrganizations/Orderer/orderers/orderer.Orderer/genesis'
cp /home/HLF_Node_Setup/genesis.block /home/HLF_Node_Setup/cryptos/ordererOrganizations/Orderer/orderers/orderer.Orderer/genesis
restart orderer container
'sudo docker compose ps -a' to check exited container in case of not showing in 'sudo docker compose ps' list


Generating the channel transaction
-----------------------------------
export CHANNEL_NAME=educationchannel  && /home/HLF_Node_Setup/bin/configtxgen -profile EducationNetwork -outputCreateChannelTx /home/HLF_Node_Setup/educationchannel.tx -channelID $CHANNEL_NAME
** educationchannel.tx file created inside main folder

Generating the channel Block 
------------------------------
** copy core.yaml file inside cryptos folder
export CHANNEL_NAME=Educationchannel  
export CORE_PEER_MSPCONFIGPATH=/home/HLF_Node_Setup/cryptos/peerOrganizations/School/users/Admin@School/msp
export CORE_PEER_ADDRESS=192.168.74.9:11010 
export CORE_PEER_LOCALMSPID="SchoolMSP"
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_TLS_ROOTCERT_FILE=/home/HLF_Node_Setup/cryptos/peerOrganizations/School/peers/peer0.School/tls/ca.crt
export FABRIC_CFG_PATH=/home/HLF_Node_Setup/cryptos

/home/HLF_Node_Setup/bin/peer channel create -o 192.168.74.9:12010 -c $CHANNEL_NAME -f /home/HLF_Node_Setup/educationchannel.tx --tls --cafile /home/HLF_Node_Setup/cryptos/ordererOrganizations/Orderer/orderers/orderer.Orderer/msp/tlscacerts/tlsca.Orderer-cert.pem
**educationchannel.block file is created in main folder

Joining the nodes to channel
-------------------------------
(add server id and respective ports)
export CHANNEL_NAME=educationchannel  
export CORE_PEER_MSPCONFIGPATH=/home/HLF_Node_Setup/cryptos/peerOrganizations/School/users/Admin@School/msp
export CORE_PEER_ADDRESS=192.168.74.9:11012
export CORE_PEER_LOCALMSPID="SchoolMSP"
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_TLS_ROOTCERT_FILE=/home/HLF_Node_Setup/cryptos/peerOrganizations/School/peers/peer0.School/tls/ca.crt
export FABRIC_CFG_PATH=/home/HLF_Node_Setup/cryptos

export CHANNEL_NAME=educationchannel  
export CORE_PEER_MSPCONFIGPATH=/home/HLF_Node_Setup/cryptos/peerOrganizations/College/users/Admin@College/msp
export CORE_PEER_ADDRESS=192.168.74.9:11010
export CORE_PEER_LOCALMSPID="CollegeMSP"
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_TLS_ROOTCERT_FILE=/home/HLF_Node_Setup/cryptos/peerOrganizations/College/peers/peer0.College/tls/ca.crt
export FABRIC_CFG_PATH=/home/HLF_Node_Setup/cryptos

export CHANNEL_NAME=educationchannel  
export CORE_PEER_MSPCONFIGPATH=/home/HLF_Node_Setup/cryptos/peerOrganizations/Uni/users/Admin@Uni/msp
export CORE_PEER_ADDRESS=192.168.74.9:11011
export CORE_PEER_LOCALMSPID="UniMSP"
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_TLS_ROOTCERT_FILE=/home/HLF_Node_Setup/cryptos/peerOrganizations/Uni/peers/peer0.Uni/tls/ca.crt
export FABRIC_CFG_PATH=/home/HLF_Node_Setup/cryptos

/home/HLF_Node_Setup/bin/peer channel join -b /home/HLF_Node_Setup/educationchannel.block

Check peers are joining in channel
----------------------------------
Sudo docker exec -it peer0.College sh
> peer channel list

Check channel name is there in peer list