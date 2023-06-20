# IEBS universidades

This Hyperledger Fabric network consists of the following components:

- 3 peer nodes, whose ledgers use couchdb.
- 1 Raft orderer node.
- All nodes have their own CA.

The genesis block configuration will be given by:

- A block time of 2 seconds.
- A block with a maximum of 10 transactions.
- A block, which, at most, will have 99Mb.

## Instalation instructions

```sh
# Before we begin we must set some environment variables
export PATH=${PWD}/../bin:${PWD}:$PATH
export FABRIC_CFG_PATH=${PWD}/../config

# We start by deleting the traces of previously created networks
# 🔥 Caution, the following Docker commands delete all Docker data. You should not run them if you have other containers running that you do not want to delete.
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
docker volume prune
docker network prune
rm -rf organizations/peerOrganizations
rm -rf organizations/ordererOrganizations
sudo rm -rf organizations/fabric-ca/madrid/
sudo rm -rf organizations/fabric-ca/bogota/
sudo rm -rf organizations/fabric-ca/berlin/
sudo rm -rf organizations/fabric-ca/ordererOrg/
rm -rf channel-artifacts/
mkdir channel-artifacts

# We start the CA and register the certificates for each node
docker-compose -f docker/docker-compose-ca.yaml up -d
. ./organizations/fabric-ca/registerEnroll.sh && createOrderer
. ./organizations/fabric-ca/registerEnroll.sh && createMadrid
. ./organizations/fabric-ca/registerEnroll.sh && createBogota
. ./organizations/fabric-ca/registerEnroll.sh && createBerlin

# We started the university network
docker-compose -f docker/docker-compose-universidades.yaml up -d

# We bind nodes to the channel universidadeschannel
export FABRIC_CFG_PATH=${PWD}/configtx
configtxgen -profile UniversidadesGenesis -outputBlock ./channel-artifacts/universidadeschannel.block -channelID universidadeschannel
export FABRIC_CFG_PATH=${PWD}/../config
export ORDERER_CA=${PWD}/organizations/ordererOrganizations/universidades.com/orderers/orderer.universidades.com/msp/tlscacerts/tlsca.universidades.com-cert.pem
export ORDERER_ADMIN_TLS_SIGN_CERT=${PWD}/organizations/ordererOrganizations/universidades.com/orderers/orderer.universidades.com/tls/server.crt
export ORDERER_ADMIN_TLS_PRIVATE_KEY=${PWD}/organizations/ordererOrganizations/universidades.com/orderers/orderer.universidades.com/tls/server.key
osnadmin channel join --channelID universidadeschannel --config-block ./channel-artifacts/universidadeschannel.block -o localhost:7053 --ca-file "$ORDERER_CA" --client-cert "$ORDERER_ADMIN_TLS_SIGN_CERT" --client-key "$ORDERER_ADMIN_TLS_PRIVATE_KEY"

export CORE_PEER_TLS_ENABLED=true
export PEER0_MADRID_CA=${PWD}/organizations/peerOrganizations/madrid.universidades.com/peers/peer0.madrid.universidades.com/tls/ca.crt
export CORE_PEER_LOCALMSPID="MadridMSP"
export CORE_PEER_TLS_ROOTCERT_FILE=$PEER0_MADRID_CA
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/madrid.universidades.com/users/Admin@madrid.universidades.com/msp
export CORE_PEER_ADDRESS=localhost:7051
peer channel join -b ./channel-artifacts/universidadeschannel.block

export PEER0_BOGOTA_CA=${PWD}/organizations/peerOrganizations/bogota.universidades.com/peers/peer0.bogota.universidades.com/tls/ca.crt
export CORE_PEER_LOCALMSPID="BogotaMSP"
export CORE_PEER_TLS_ROOTCERT_FILE=$PEER0_BOGOTA_CA
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/bogota.universidades.com/users/Admin@bogota.universidades.com/msp
export CORE_PEER_ADDRESS=localhost:9051
peer channel join -b ./channel-artifacts/universidadeschannel.block

export PEER0_BERLIN_CA=${PWD}/organizations/peerOrganizations/berlin.universidades.com/peers/peer0.berlin.universidades.com/tls/ca.crt
export CORE_PEER_LOCALMSPID="BerlinMSP"
export CORE_PEER_TLS_ROOTCERT_FILE=$PEER0_BERLIN_CA
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/berlin.universidades.com/users/Admin@berlin.universidades.com/msp
export CORE_PEER_ADDRESS=localhost:2051
peer channel join -b ./channel-artifacts/universidadeschannel.block
```
