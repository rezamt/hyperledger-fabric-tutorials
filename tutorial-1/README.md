## Getting Started

These instructions have been verified to work against the version "1.0.0-beta" tagged docker images and the pre-compiled setup utilities within the supplied tar file. The following picture shows what we are going to build during the tutorial.

![alt Highlevel](https://raw.githubusercontent.com/rezamt/hyperledger-fabric-tutorials/master/tutorial-1/docs/HLD.png)

### Before begin 

1- Makesure Goland has been installed and configured correctly. Worth to check your GoPath



``` bash
go version

:: Output
go version go1.8.3 darwin/amd64


echo $GOPATH

:: Output
/Users/reza/go


brew install gnu-tar --with-default-names
brew install libtool

```

2- Checkout Fabric Project from Github under your GOPATH

```
cd $GOPATH
mkdir -p $GOPATH/src/github.com/hyperledger
cd $GOPATH/src/github.com/hyperledger
git clone https://gerrit.hyperledger.org/r/#/admin/projects/fabric

```

### Building Fabric tools  

Now, lets compile and build the following tools:

- github.com/hyperledger/fabric/common/configtx/tool/configtxgen
- github.com/hyperledger/fabric/common/tools/cryptogen
- github.com/hyperledger/fabric/common/tools/configtxlator
- github.com/hyperledger/fabric/peer

Each tool consumes a configuration yaml file (you will see in example directory), within which we specify the topology of our network (cryptogen) and the location of our certificates for various
configuration operations (configtxgen).  Once the tools have been successfully run,we are able to launch our network.  More detail on the tools and the structure of the network will be provided later in this document. 

```
cd $GOPATH/src/github.com/hyperledger/fabric
make release

ls -rtl release/darwin-amd64/bin

:: Output

-rwxr-xr-x  1 reza  staff  14527172 22 Jun 23:05 configtxgen            # Utility for manipulating fabric channel configuration.
-rwxr-xr-x  1 reza  staff   7280884 22 Jun 23:05 cryptogen              # Utility for generating Hyperledger Fabric key material
-rwxr-xr-x  1 reza  staff  16018660 22 Jun 23:05 configtxlator          # Utility for generating Hyperledger Fabric channel configurations
-rwxr-xr-x  1 reza  staff  22647108 22 Jun 23:05 peer                   # Fabric CLI Utility tool to intract with Channel, Peer, Chaincode 

```

## Cryptogen Tool (cryptogen)

We will use the cryptogen tool to generate the cryptographic material (x509 certs)
for our various network entities.  The certificates are based on a standard PKI
implementation where validation is achieved by reaching a common trust anchor.

### How does it work?

Cryptogen consumes a file - ``crypto-config.yaml`` - that contains the network
topology and allows us to generate a library of certificates for both the
Organizations and the components that belong to those Organizations.  Each
Organization is provisioned a unique root certificate (``ca-cert``), that binds
specific components (peers and orderers) to that Org.  Transactions and communications
within Fabric are signed by an entity's private key (``keystore``), and then verified
by means of a public key (``signcerts``).  You will notice a "count" variable within
this file.  We use this to specify the number of peers per Organization; in our
case it's two peers per Org.  The rest of this template is extremely
self-explanatory.

### Result of Execution

After we run the tool, the certs will be parked in a folder titled ``crypto-config``.

## Configuration Transaction Generator (configtxgen)

The [configtxgen tool](https://github.com/hyperledger/fabric/blob/master/docs/source/configtxgen.rst)
is used to create four artifacts: orderer **bootstrap block**, fabric
**channel configuration transaction**, and two **anchor peer transactions** - one
for each Peer Org.

The orderer block is the genesis block for the ordering service, and the
channel transaction file is broadcast to the orderer at channel creation
time.  The anchor peer transactions, as the name might suggest, specify each
Org's anchor peer on this channel.

### How does it work?

Configtxgen consumes a file - ``configtx.yaml`` - that contains the definitions
for the sample network. There are three members - one Orderer Org (``OrdererOrg``)
and two Peer Orgs (``Org1`` & ``Org2``) each managing and maintaining two peer nodes.
This file also specifies a consortium - ``SampleConsortium`` - consisting of our
two Peer Orgs.  Pay specific attention to the "Profiles" section at the top of
this file.  You will notice that we have two unique headers. One for the orderer genesis
block - ``TwoOrgsOrdererGenesis`` - and one for our channel - ``TwoOrgsChannel``.
These headers are important, as we will pass them in as arguments when we create
our artifacts.  This file also contains two additional specifications that are worth
noting.  Firstly, we specify the anchor peers for each Peer Org
(``peer0.org1.example.com`` & ``peer0.org2.example.com``).  Secondly, we point to
the location of the MSP directory for each member, in turn allowing us to store the
root certificates for each Org in the orderer genesis block.  This is a critical
concept. Now any network entity communicating with the ordering service can have
its digital signature verified.

### Result of Execution

These configuration transactions will bundle the crypto material for the
participating members and their network components and output an orderer
genesis block and three channel transaction artifacts. These artifacts are
required to successfully bootstrap a Fabric network and create a channel to
transact upon.


### Downloading Fabric Docker Images
Lets foucs on "get-docker-images.sh" to download all images that we need to practice in this tutorial and the next tutorials.

```
cd $GOPATH/src/github.com/hyperledger/fabric
make docker

:: Output
ls -rtl $GOPATH/src/github.com/hyperledger/fabric/build/image

    drwxr-xr-x  5 reza  staff  170 23 Jun 00:16 tools
    drwxr-xr-x  5 reza  staff  170 23 Jun 00:23 ccenv
    drwxr-xr-x  5 reza  staff  170 23 Jun 00:27 javaenv
    drwxr-xr-x  5 reza  staff  170 23 Jun 00:27 peer
    drwxr-xr-x  5 reza  staff  170 23 Jun 00:27 orderer
    drwxr-xr-x  5 reza  staff  170 23 Jun 00:27 buildenv
    drwxr-xr-x  5 reza  staff  170 23 Jun 00:28 testenv
    drwxr-xr-x  5 reza  staff  170 23 Jun 00:29 zookeeper
    drwxr-xr-x  5 reza  staff  170 23 Jun 00:29 kafka
    drwxr-xr-x  5 reza  staff  170 23 Jun 00:32 couchdb



docker images hyperledger/*

:: Ouput
    REPOSITORY                     TAG                                   IMAGE ID            CREATED              SIZE
    hyperledger/fabric-couchdb     latest                                58edd590915e        About a minute ago   1.48GB
    hyperledger/fabric-couchdb     x86_64-1.0.0-rc1-snapshot-279fe2690   58edd590915e        About a minute ago   1.48GB
    hyperledger/fabric-kafka       latest                                4b04b750a8e7        4 minutes ago        1.3GB
    hyperledger/fabric-kafka       x86_64-1.0.0-rc1-snapshot-279fe2690   4b04b750a8e7        4 minutes ago        1.3GB
    hyperledger/fabric-zookeeper   latest                                7434ba8dbb3b        4 minutes ago        1.31GB
    hyperledger/fabric-zookeeper   x86_64-1.0.0-rc1-snapshot-279fe2690   7434ba8dbb3b        4 minutes ago        1.31GB
    hyperledger/fabric-testenv     latest                                2e711dff7f23        5 minutes ago        1.41GB
    hyperledger/fabric-testenv     x86_64-1.0.0-rc1-snapshot-279fe2690   2e711dff7f23        5 minutes ago        1.41GB
    hyperledger/fabric-buildenv    latest                                1b245255bb1a        6 minutes ago        1.32GB
    hyperledger/fabric-buildenv    x86_64-1.0.0-rc1-snapshot-279fe2690   1b245255bb1a        6 minutes ago        1.32GB
    hyperledger/fabric-orderer     latest                                543c5d8a7a84        6 minutes ago        179MB
    hyperledger/fabric-orderer     x86_64-1.0.0-rc1-snapshot-279fe2690   543c5d8a7a84        6 minutes ago        179MB
    hyperledger/fabric-peer        latest                                f87ebdef0fca        6 minutes ago        182MB
    hyperledger/fabric-peer        x86_64-1.0.0-rc1-snapshot-279fe2690   f87ebdef0fca        6 minutes ago        182MB
    hyperledger/fabric-javaenv     latest                                c285eb861bff        6 minutes ago        1.42GB
    hyperledger/fabric-javaenv     x86_64-1.0.0-rc1-snapshot-279fe2690   c285eb861bff        6 minutes ago        1.42GB
    hyperledger/fabric-ccenv       latest                                5957ae6c452f        10 minutes ago       1.29GB
    hyperledger/fabric-ccenv       x86_64-1.0.0-rc1-snapshot-279fe2690   5957ae6c452f        10 minutes ago       1.29GB
    hyperledger/fabric-tools       latest                                990007d59a6f        17 minutes ago       1.32GB
    hyperledger/fabric-tools       x86_64-1.0.0-rc1-snapshot-279fe2690   990007d59a6f        17 minutes ago       1.32GB
    hyperledger/fabric-buildenv    x86_64-1.0.0-rc1-snapshot-5acdf090    d23aa70441c6        17 minutes ago       1.32GB
    hyperledger/fabric-ca          x86_64-1.0.0-beta                     e549e8c53c2e        17 minutes ago       238MB
    hyperledger/fabric-baseimage   x86_64-0.3.1                          9f2e9ec7c527        17 minutes ago       1.27GB
    hyperledger/fabric-baseos      x86_64-0.3.1                          4b0cab202084        17 minutes ago       157MB

# You can build a subset of above images using the following commnads:

e.g. make peer-docker 
#   - peer-docker[-clean] - ensures the peer container is available[/cleaned]
#   - orderer-docker[-clean] - ensures the orderer container is available[/cleaned]
#   - tools-docker[-clean] - ensures the tools container is available[/cleaned]


```

## Building our End-2-End Fabric Trading Application
Now we are going to start with the first example e2e_cli, which is my favorit for learning how different components are wokring together.

![alt Highlevel](https://raw.githubusercontent.com/rezamt/hyperledger-fabric-tutorials/master/tutorial-1/docs/HLD.png)

### Preparation
```
cd $GOPATH/src/github.com/hyperledger/fabric/examples
cp -R e2e_cli e2e_demo   # for the purpose of this tutorials
cd e2e_demo
tree

├── base                                 # Docker base template file 
│   ├── docker-compose-base.yaml
│   └── peer-base.yaml
├── channel-artifacts                    # Channel artifacts that will be generated and saved in this directory
├── configtx.yaml                        
├── crypto-config.yaml
├── docker-compose-cli.yaml
├── docker-compose-couch.yaml
├── docker-compose-e2e-template.yaml
├── docker-compose-e2e.yaml
├── download-dockerimages.sh
├── end-to-end.rst
├── examples
│   └── chaincode
│       └── go
│           └── chaincode_example02
│               └── chaincode_example02.go
├── generateArtifacts.sh                # Generates channel artifacts and stores them under channel-artifacts folder
├── network_setup.sh                    # Fully automated script to stop/start a Fabric network 
└── scripts                               
    └── script.sh                        # Step by Step 


```
### Generating Orderers Service Node and Channel Peers Certificates (Identities)

Choose a channel name: my-channel and make sure you are under e2e_demo folder.

```
# Set OS Architecture Type env variable

os_arch=$(echo "$(uname -s)-amd64" | awk '{print tolower($0)}')

# Copy tools binary folder under release to bin in e2e_demo

cp -R ./../../release/$os_arch/bin .

#
# Generate certificates using cryptogen tool for (org1.example.com and org2.example.com)
#

./bin/cryptogen generate --config=./crypto-config.yaml

# which generates a set of certificates for both orderer and peers under crypto-config folder

tree crypto-config

├── ordererOrganizations
│   └── example.com
│       ├── ca
│       ├── msp
│       ├── orderers
│       │   └── orderer.example.com
...

└── peerOrganizations
    ├── org1.example.com
    │   ├── ca
    │   ├── msp
    │   │   ├── admincerts
    │   │   ├── cacerts
    │   │   └── tlscacerts
...
    │   ├── peers
    │   │   ├── peer0.org1.example.com

    │   └── users
    │       ├── Admin@org1.example.com
    │       └── User1@org1.example.com
...
    └── org2.example.com
    │   ├── ca
    │   ├── msp
    │   │   ├── admincerts
    │   │   ├── cacerts
    │   │   └── tlscacerts
...
    │   ├── peers
    │   │   ├── peer0.org1.example.com
...
    │   └── users
    │       ├── Admin@org1.example.com
    │       └── User1@org1.example.com

# 

```

### Generating Channel Configuration and Gensis Block
Next, we need to tell the configtxgen tool where to look for the configtx.yaml file that it needs to ingest.  We will tell it look in our present working directory:

Please make sure you are under e2e_demo folder.
```
CHANNEL_ID=my-channel
FABRIC_CFG_PATH=$PWD

# Create the orderer genesis block:

./bin/configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block

# Create the channel transaction artifact:

./bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_ID

```

### Define the Anchor Peers for both Organizations Org1 & org2 (e.g. my-channel):

```
./bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_ID -asOrg Org1MSP

./bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID $CHANNEL_ID -asOrg Org2MSP
```

Note you should now be able to see the following information under:

```
ls -rtl ./channel-artifacts

-rw-r--r--  1 reza  staff  9040 23 Jun 22:53 genesis.block      # First genesis block
-rw-r--r--  1 reza  staff   371 23 Jun 22:55 channel.tx         # First channel transaction
-rw-r--r--  1 reza  staff   252 23 Jun 22:57 Org1MSPanchors.tx  # Org1 Anchor Peer transaction
-rw-r--r--  1 reza  staff   252 23 Jun 22:58 Org2MSPanchors.tx  # Org2 Anchor Peer transaction

```

### Running the end-to-end test with Docker
```
# Updating docker network name 

sed -i -e 's/e2ecli_default/e2edemo_default/' $PWD/base/peer-base.yaml

# Runing Docker Compose

CHANNEL_NAME=my-channel TIMEOUT=10000000 docker-compose -f docker-compose-cli.yaml up -d

# If you set a moderately high TIMEOUT value, then you will see your cli
container as well.

# Tailing Fabric Cli Docker log in order to see the execution of commands
docker logs -f cli

# Tailing Fabric orderer Docker log in order to see the creation of gensis block
docker logs -f orderer.example.com


# The chaincode logs?
docker logs dev-peer0.org1.example.com-mycc-1.0
docker logs dev-peer0.org2.example.com-mycc-1.0

```

## What happend ? Commands get executed in which ordder? 

A script - script/script.sh - is baked inside the CLI container. The script uses peer tool against the supplied channel name and uses the channel.tx (channel-artifacts) file for channel configuration.

```
# make sure you are inside examples/e2e_demo folder.

./bin/peer

:: Output
Usage
  peer [flags]
  peer [command]

Available Commands:
  chaincode   Operate a chaincode: install|instantiate|invoke|package|query|signpackage|upgrade.
  channel     Operate a channel: create|fetch|join|list|update.
  logging     Log levels: getlevel|setlevel|revertlevels.
  node        Operate a peer node: start|status.
  version     Print fabric peer version.

Flags:
  -h, --help                       help for peer
      --logging-level string       Default logging level and overrides, see core.yaml for full syntax
      --test.coverprofile string   Done (default "coverage.cov")
  -v, --version                    Display current version of fabric peer server

Use "peer [command] --help" for more information about a command.

```

Here are the commands executed - pesudo code (Commands need environment variables)

```
#
# Create Channel (only do it for the first peer0.org1.example.com)
peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME 
            -f ./channel-artifacts/channel.tx 
            --tls $CORE_PEER_TLS_ENABLED 
            --cafile $ORDERER_CA 


#
# Joining All peers to the channel
for peer in [org1.peer0, org1.peer1, org2.peer0, org2.peer1]; do
    peer channel join -b $CHANNEL_NAME.block
end

#
# Update Anchor Peers on the channel
# The anchor peers for Org1MSP (peer0.org1.example.com) and Org2MSP (peer0.org2.example.com) are then updated. 
# We do this by passing the Org1MSPanchors.tx and Org2MSPanchors.tx artifacts to the ordering service along with the name of our channel.

peer channel update -o ORDERER-SERVE:PORT
             -c $CHANNEL_NAME 
             -f ./channel-artifacts/${CORE_PEER_LOCALMSPID}anchors.tx 

#             
# Install Chaincode on Peers participating in block chain ( V1.0 means HyperLedgerFabric v1.0 or you can do all on v0.6 if you want)

for peer in [org1.peer0, org1.peer1, org2.peer0, org2.peer1]; do
	peer chaincode install -n $SMART-CONTRACT-ID -v 1.0 -p YOUR-FABRIC-SMART-CONTRACT-CODE-PATH
end

#
# Instantiate chaincode on Peers
for peer in [org1.peer0, org1.peer1, org2.peer0, org2.peer1]; do
    peer chaincode instantiate -o ORDERER-SERVE:PORT -C $CHANNEL_NAME -n $SMART-CONTRACT-ID -v 1.0 -c 'SMART CONTRACT JSON FORMATTED ARGUMENT' -P "OR	('Org1MSP.member','Org2MSP.member')"
end

#
# Invoking Smart contract
peer chaincode invoke -o ORDERER-SERVE:PORT  --cafile $ORDERER_CA -C $CHANNEL_NAME -n $SMART-CONTRACT-ID -c 'SMART CONTRACT JSON FORMATTED ARGUMENT'

```

## All in one

The network_setup.sh script literally does it all. It calls generateArtifacts.sh to exercise the cryptogen and configtxgen tools, followed by script.sh which launches the network, joins peers to a generated channel and then drives transactions. If you choose not to supply a channel ID, then the script will use a default name of mychannel. The cli timeout parameter is an optional value; if you choose not to set it, then your cli container will exit upon conclusion of the script.

```
# START
./network_setup.sh up $CHANNEL_ID 1000000  # network_setup.sh up <channel-ID> <timeout-value>

# STOP
./network_setup.sh down $CHANNEL_ID

```


## Removing Fabric Containers

```
docker rmi -f $(docker images hyperledger/* -q)

```

## REFERENCES

- [Fabric Tools and Sources Documentation](https://github.com/hyperledger/fabric/tree/master/docs/source)
- [Channel Configuration (configtxgen)](https://github.com/hyperledger/fabric/blob/master/docs/source/configtxgen.rst)
- [Channel Definition](https://github.com/hyperledger/fabric/blob/master/docs/source/channels.rst)
- [Chain Code](https://github.com/hyperledger/fabric/blob/master/docs/source/chaincode.rst)
- [Policies in Hyperledger Fabric](https://github.com/hyperledger/fabric/blob/master/docs/source/policies.rst)
- [Demo Transcript](https://github.com/hyperledger/fabric/blob/master/docs/source/getting_started.rst)

- [Online Doc](https://rezamtfabric.readthedocs.io/en/latest/)
