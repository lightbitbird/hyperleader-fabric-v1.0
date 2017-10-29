
Hyperledger Fabric v1.0 環境構築手順

- v1.0-alpha2 2017/06/18 現在 -

http://hyperledger-fabric.readthedocs.io/en/latest/getting_started.html#crypto-generator 参照

以下はMac OSにインストールしたため、必要に応じてLinux用に置き換えること


▪️Homebrewインストール
Homebrew
http://qiita.com/omega999/items/6f65217b81ad3fffe7e6


▪️Gitインストール
http://qiita.com/furusin_oriver/items/974a7b7fb8c56ad88d6e


▪️GOインストールする
#brew よりインストール
brew install go

#作業スペース GOパス設定
#/usr/local/bin/go -> ../Cellar/go/1.8.1/bin/go
vim ~/.bashrc
export GOPATH=/usr/local/Cellar/go/1.8.1/libexec


▪️Hyperledger Fabric環境構築

#fabricをクローン
mkdir -p $GOPATH/src/github.com/hyperledger
cd $GOPATH/src/github.com/hyperledger/
git clone https://github.com/hyperledger/fabric.git
cd fabric


▪️v1.0 Getting Started
#環境構築の前にdocker コンテナやイメージを初期化しておく
#コンテナを削除
docker rm $(docker ps -aq)

#docker images削除
docker rmi $(docker images -f 'dangling=true' -q)
#全てのdocker imagesを削除する場合
docker rmi -f `docker images -aq`


#認証とネットワーク環境のセットアップ
mkdir fabric-sample
cd fabric-sample
curl -sSL https://goo.gl/LQkuoh | bash
#fabric-sample配下に生成されたbootstrapを実行
./bootstrap—1.0.0-alpha.sh

#cryptogen & configtxgen、ネットワークをまとめて構築したい場合は generateArtifacts.sh
./generateArtifacts.sh <channel ID>

#docker-compose-cli.yamlのscriptを実行するコマンドをコメント
vim docker-compose-cli.yaml
woring_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
#command: /bin/bash -C './scripts/script.sh ${CHANNEL_NAME}; sleep $tIMEOUT'
volumes

#networkを開始
CHANNEL_NAME=<channel ID> TIMEOUT=60 docker-compose -f docker-compose-cli.yaml up -d

#環境の違いによる設定変更をまとめておく
#Environment variables for PEER0
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
CORE_PEER_ADDRESS=peer0.org1.example.com:7051
CORE_PEER_LOCALMSPID="Org1MSP"
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt

#Dockerに入りChannelを作成
docker exec -it cli bash
root@0d78bb69300d:/opt/gopath/src/github.com/hyperledger/fabric/peer#
export CHANNEL_NAME=<channel ID>
#channel作成
peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/cacerts/ca.example.com-cert.pem

#channel参加
peer channel join -b <channel ID>

#chaincodeのInstall & 初期化
#install
peer chaincode install -n mycc -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02

#初期化
#be sure to replace the $CHANNEL_NAME environment variable
#if you did not install your chaincode with a name of mycc, then modify that argument as well
peer chaincode instantiate -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/cacerts/ca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 -c '{"Args":["init","a", "100", "b","200"]}' -P "OR ('Org1MSP.member','Org2MSP.member’)"

#Query
#be sure to set the -C and -n flags appropriately
peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'

#Invoke
peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/cacerts/ca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -c '{"Args":["invoke","a","b","10"]}'

#Query
peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}’

#aの値10がbに渡り、結果a=90となる
#Query Result : 90



