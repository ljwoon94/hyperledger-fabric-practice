# hyperledger-fabric-practice
하이퍼레저 패브릭 튜토리얼 공부


# 하이퍼레저 패브릭이란?
하이퍼레저 패브릭은 허가형 프라이빗 블록체인(Permissioned Private Blockchain)의 형태를 가진다. 누구나 자유롭게 참여가 가능한 기존의 퍼블릭 블록체인과 달리, 하이퍼레저 패브릭에서는 인증 관리 시스템에 의해 허가된 사용자만이 블록체인 네트워크에 참여할 수 있다. 따라서 패브릭 네트워크에 참여한 노드들은 이미 시스템에 의해 허가된, 신뢰를 가진 노드로 볼 수 있고 퍼블릭 블록체인에서 사용하는 악의적인 노드를 검증하기 위한 복잡한 합의 알고리즘 등을 필요로 하지 않는다. 단지 원장에 접근하려는 사용자가 허가된 노드인가, 그러한 권한이 있는가, 트랜잭션이 제대로 구성되어 있는가를 검증하는 정도로 충분하다. 물론 필요에 의해 원하는 합의 알고리즘을 네트워크 내에서 선택적으로 사용할 수도 있다.
 패브릭에서는 모든 노드가 동일한 원장으로 정보를 공유할 수 있고, 비즈니스 목적에 맞게 공유하고자 하는 노드 간에만 별도의 원장을 생성하는 것도 가능하다. 기존의 블록체인 네트워크에서는 참여한 모든 노드에게 원장에 기록되어 있는 정보가 공유되었다. 하지만 기업 입장에서 비즈니스를 하다 보면 모두에게 공유하고 싶지 않은 민감한 정보들이 생기기 마련이다. 하이퍼레저 패브릭은 네트워크 내에서 목적에 맞는 별도의 원장을 생성할 수 있는 채널(Channel)을 제공함으로써 기업이 사용하기 용이하도록 고안되었다.


# 하이퍼레저 패브릭 구성요소
(Hyperledger Fabric Component)

하이퍼레저 패브릭의 특징을 구현하기 위한 구성요소에는 크게 분산원장, 체인코드, peer, orderer가 있다.

 블록체인 기술의 핵심인 분산원장(Distributed Ledger)이 있다. 공유하고자 하는 데이터의 변화를 모두 기록해둔 것이 원장이다. 원장은 현재의 상태를 저장해 놓은 데이터베이스인 월드 스테이트(World State)와 상태변화에 대한 모든 로그 기록이 저장 되어있는 블록체인(Blockchain) 부분으로 나뉘어있다. 추가로 원장에 새로운 내용을 업데이트 하거나 기존의 내용을 읽어 오기 위해 필요한 것이 바로 체인코드(Chaincode)이다.

 이러한 원장과 체인코드를 관리하며 패브릭 네트워크를 구성하는 노드를 peer라 부른다. 패브릭 네트워크 참여자들은 peer에 설치되어 있는 체인코드 실행 요청을 통해 peer에 저장된 원장에 데이터를 읽거나 쓸 수 있다. 이러한 요청은 보통 사용자의 편의를 위해 체인코드와 함께 개발되는 dapp을 통해 이루어진다. 체인코드 실행을 요청하는 트랜잭션이 발생하면 3단계(execution - ordering - validation)의 과정을 거쳐 원장에 기록되고 사용자에게 결과를 반환한다. peer는 수행하는 역할에 따라 크게 다음과 같이 4가지로 구분된다.

Endorsing peer - 체인코드 시뮬레이션을 통해 트랜잭션이 적절한지 판단하는 역할을 한다. 3단계의 과정 중 execution에 해당한다.

Committing peer - 모든 peer가 수행하는 역할로, 최신 블록에 대한 검증을 한다. 위의 3단계의 과정 중 validation에 해당한다.

Anchor peer - 다른 조직과의 통신을 위해 다른 조직의 peer와 통신하는 역할을 한다.

Leader peer - orderer와 연결되어 최신 블록을 전달받아 조직 내 다른 peer들에게 전송하는 역할을 한다.  

Endorsing peer들이 시뮬레이션을 통해 적절하다고 판단한 트랜잭션들을 모아서 정렬한 후 실제 블록을 생성하는 노드를 orderer라 한다. 현재 하이퍼레저 패브릭에서 orderer가 트랜잭션의 순서를 정렬하는 방법에도 solo와 kafka방식으로 2가지가 존재한다. solo방식은 보통 테스트용으로 orderer 하나가 정렬 및 블록 생성의 모든 과정을 담당하는 방식이다. kafka방식은 분산 메시징 시스템인 kafka cluster를 통해 orderer가 트랜잭션을 정렬하고 블록을 생성하는 방식이다. orderer는 블록을 생성한 후 자신에게 연결되어 있는 leader peer들에게 블록을 전달하고, leader peer들이 다시 자신이 속한 채널의 peer들에게 블록을 전달하면 peer들은 블록을 검증한 후 자신의 원장에 추가시키게 된다. 패브릭에서는 체인코드 실행을 요청하는 트랜잭션부터 원장에 기록되는 과정을 통틀어 합의라고 부른다.

 위에서 패브릭은 허가된 사용자만이 참여할 수 있는 허가형 블록체인이라 하였다. 패브릭에서는 사용자의 권한 및 인증을 위해 MSP(Membership Service Provider)라는 인증 관리 시스템을 사용하는데, 여기에는 네트워크 내 노드의 역할과 권한 등이 정의되어 있다.

 이러한 MSP를 발급하고 관리하는 역할을 하는 기관을 CA(Certificate Authority)라고 한다. 사용자를 인증해 주는 것은 중요한 역할이기 때문에 CA는 보통 신뢰 있는 기관이 담당하는데, 하이퍼레저 패브릭에서는 Fabric-CA 노드가 그 역할을 수행한다.

 하이퍼레저 패브릭에서는 CA노드를 통해 1차적으로 사용자의 서명과 권한 등을 확인하고, peer를 통해 원장에 기록되기 전에 보증 정책(Endorsement Policy)을 준수하는지 확인하는 과정을 거친다. 보증 정책은 보통 해당 트랜잭션이 지정된 peer들의 허가를 받아야 한다는 내용인데, 원장을 공유하는 채널별로 참여자들은 다양한 방식으로 보증 정책을 설정할 수 있다.
 
 
--------------------------------------------------------

# 튜토리얼
## 1. 사전 준비 (환경설정)

하이퍼레저 패브릭에서 제공해주는 하이퍼레저 패브릭 샘플을
시작하기전 필요한 사전 작업을 해야한다.

순서대로
WSL2 설치 (Ubuntu 20.04.2 LTS)
Docker Desktop 최신버전
go, jq, curl ,git, nvm 을 우분투에다 설치
sudo apt install build-essential

fabric 과 fabric samples 설치
우분투에 경로 생성
```
$ mkdir -p $HOME/go/src/github.com/<your_github_userid>
$ cd $HOME/go/src/github.com/<your_github_userid>
``` 

curl로 패브릭 샘플 내려받기
```
$ curl -sSL https://bit.ly/2ysbOFE | bash -s
```
-------------------------------------
## 2-1. 테스트 네트워크 실행
 
 테스트 네트워크 불러오기
 
 ```
 cd ~/go/src/github.com/ljwoon94/fabric-samples/test-network
 ```

![image](https://user-images.githubusercontent.com/68358404/121637686-bee4d280-cac4-11eb-8dee-31650f9fcb3c.png)

경로에 파일 확인 후 스크립트 실행

```
ls
./network.sh up
```

![image](https://user-images.githubusercontent.com/68358404/121637975-3286df80-cac5-11eb-95b8-27ed6e0c2eed.png)

성공하면 두개의 피어노드, 하나의 오더노드로 구성된 패크릭 네트워크 생성

![image](https://user-images.githubusercontent.com/68358404/121638069-55b18f00-cac5-11eb-9411-3d2e900d11a7.png)

네트워크 구성 요소를 다시 보고 싶으면 

```
docker ps -a
```

 피어 는 모든 패브릭 네트워크의 기본 구성 요소이다. 피어는 블록 체인 원장을 저장하고 원장에 커밋되기 전에 거래를 검증한다. 피어는 블록 체인 원장에서 자산을 관리하는데 사용되는 비즈니스 로직이 포함된 스마트 계약을 실행한다.
 
 네트워크의 모든 피어는 조직에 속해야한다. 테스트 네트워크에서 각 조직(peer0.org1.example.com, peer0.org2.example.com)은 각각 하나의 피어를 운영한다.
 
 오더 노드는 클라이언트로부터 보증된 트랜잭션을 수신 한 후 트랜잭션 순서에 대한 합의에 도달 한 다음 블록에 추가한다. 그런 다음 블록은 블록 체인 원장에 블록을 추가하는 피어 노드에 배포된다.

## 2-2. 채널 만들기

 채널은 특정 네트워크 구성원 간의 개인 통신 계층이다. 채널은 채널에 초대 된 조직에서만 사용할 수 있으며 네트워크의 다른 구성원에게는 표시되지 않습다. 각 채널에는 별도의 블록 체인 원장이 있다. 초대를받은 조직은 피어를 채널에 "참여"하여 채널 원장을 저장하고 채널에서 트랜잭션을 검증한다.
 
 ./network.sh를 사용해 Org1 과 Org2 사이의 채널 만들기.
 
 createChannel -c <채널명>

```
./network.sh createChannel -c mychannel
```

![image](https://user-images.githubusercontent.com/68358404/121638874-a5dd2100-cac6-11eb-966e-67f30ec0cd2b.png)

## 2-3. 채널에서 체인코드 시작

 채널을 만든 후 스마트 계약 을 사용하여 채널 원장과 상호 작용할 수 있다. 스마트 계약(체인코드)는 블록 체인 원장의 자산을 관리하는 비즈니스 로직이 포함되어 있다. 네트워크 구성원이 실행하는 애플리케이션은 스마트 계약(체인코드)을 호출하여 원장에 자산을 생성하고 해당 자산을 변경 및 전송할 수 있다. 애플리케이션은 원장의 데이터를 읽기 위해 스마트 계약을 쿼리한다.
 
 Fabric에서 스마트 계약은 체인 코드라고하는 패키지로 네트워크에 배포된다. 체인 코드는 조직의 피어에 설치되고 채널에 배포되어 트랜잭션을 승인하고 블록 체인 원장과 상호 작용하는 데 사용할 수 있다. 체인 코드를 채널에 배포하려면 채널 구성원이 체인 코드 거버넌스를 설정하는 체인 코드 정의에 동의해야한다. 필요한 수의 조직이 동의하면 체인 코드 정의를 채널에 커밋 할 수 있으며 체인 코드를 사용할 준비가 된 것이다.
 
 이미 승인된 체인코드를 채널에 배포 해보기
 deployCC 체인코드 배포하는 명령어, -ccn 체인코드 네임, -ccp 체인코드 경로 , -ccl 체인코드 언어
 채널을 지정하지 않으면 기본 mychannel로 지정된다.
 
```
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
```

## 2-4. 네트워크와 상호작용

 테스트 네트워크를 가져온 후 peer CLI를 사용하여 네트워크와 상호작용 할 수 있다. peer CLI는 배포 스마트 계약, 업데이트 채널을 호출하거나, 설치 및 새로운 스마트 계약을 배포 할 수 있다.
 
 test-network 경로에 peer CLI를 실행 하는데 필요한, 환경변수를 등록한다.
 export PATH=${PWD}/../bin:$PATH 는 해당 바이너리 경로
 export FABRIC_CFG_PATH=$PWD/../config/ 는 core.yaml를 가리킨다.
 
 ```
export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=$PWD/../config/
 ```
 
  peer CLI를 Org1로 작동 할 수 있는 환경변수 설정
  CORE_PEER_TLS_ROOTCERT_FILE 및 CORE_PEER_MSPCONFIGPATH 환경변수에는 Org1의 암호화 소재를 가리는 organizations폴더에 있다.
  
 ```
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
 ```
 
 ![image](https://user-images.githubusercontent.com/68358404/121640140-821ada80-cac8-11eb-92c5-852930203ae7.png)

다음 명령을 실행하여 자산으로 원장을 초기화
 
 ```
 peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -c '{"function":"InitLedger","Args":[]}'

 ```

 출력 
 
 ![image](https://user-images.githubusercontent.com/68358404/121640237-a24a9980-cac8-11eb-9c8b-b697d66da84d.png)

 다음 명령을 실행하여 채널 원장에 추가 된 자산 목록을 가져온다.
 
```
peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
```

![image](https://user-images.githubusercontent.com/68358404/121640418-db830980-cac8-11eb-8321-f9c470043f96.png)

 체인 코드는 네트워크 구성원이 원장의 자산을 전송하거나 변경하려고 할 때 호출된다. 다음 명령을 사용하여 자산 전송 (기본) 체인 코드를 호출하여 원장의 자산 소유자를 변경해본다.
 
```
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -c '{"function":"TransferAsset","Args":["asset6","Christopher"]}' 
```

 출력

![image](https://user-images.githubusercontent.com/68358404/121640597-171dd380-cac9-11eb-9e4c-411f6565d292.png)

 자산 전송에 대한 보증 정책 체인코드(basic)가 Org1 및 Org2에 의해 서명되는 트랜잭션을 필요로 한다. --peerAddresses 와 --tlsRootCertFiles를 사용하여 각 피어(peer0.org1.example.com, peer0.org2.example.com) 에 대한 TLS 인증서도 참조해야한다.  
 
 Org2로 작동하도록 다음 환경 변수를 설정
 
```
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051 
```

![image](https://user-images.githubusercontent.com/68358404/121641213-dd010180-cac9-11eb-9cec-4287f4adb887.png)


이제 자산 전송 체인코드(basic)를 쿼리 할 수 있다.

```
peer chaincode query -C mychannel -n basic -c '{"Args":["ReadAsset","asset6"]}'
```

 출력

![image](https://user-images.githubusercontent.com/68358404/121641251-ea1df080-cac9-11eb-974d-1bd0a54e4db0.png)

## 2-5. 네트워크 중단

```
./network.sh down
```

--------------------------------

## 3-1. 채널에 스마트 계약 배포

 Hyperledger Fabric에서 스마트 계약은 체인 코드라고하는 패키지에 배포된다. 트랜잭션을 검증하거나 원장을 쿼리하려는 조직은 피어에 체인 코드를 설치해야합니다. 채널에 조인된 피어에 체인 코드가 설치되면 채널 구성원은 체인 코드를 채널에 배포하고 체인 코드의 스마트 계약을 사용하여 채널 원장에서 자산을 생성하거나 업데이트 할 수 있다.

 체인 코드는 Peer chaincode lifecycle라는 프로세스를 사용하여 채널에 배포됩니다. Fabric 체인 코드 라이프 사이클을 사용하면 여러 조직에서 체인 코드를 사용하여 트랜잭션을 생성하기 전에 작동 방식에 동의 할 수 있습니다. 피어 라이프 사이클 체인 코드 명령 을 사용해 패브릭 테스트 네트워크의 채널에 체인 코드를 배포하는 방법을 배울 수 있습니다 

## 3-2. 네트워크 시작

```
cd fabric-samples/test-network
./network.sh down
./network.sh up createChannel
```

2개의 peer 노드와 한개의 orderer 노드 생성

![image](https://user-images.githubusercontent.com/68358404/121764734-0a5cb680-cb81-11eb-95f2-c0379afab0ca.png)


## 3-3. Logspout (네트워크 로그 보기)

경로에 있는 monitordocker.sh 복사
fabric_test 네트워크 로그 감시

```
cp ../commercial-paper/organization/digibank/configuration/cli/monitordocker.sh 
./monitordocker.sh fabric_test
```

![image](https://user-images.githubusercontent.com/68358404/121764817-bbfbe780-cb81-11eb-9502-0eeb5456c72e.png)

## 3-4. 스마트 계약 패키징 (자바스크립트)

피어에 설치하기 전에 체인 코드를 패키징 해야한다.

터미널 새로 하나 킨다. 경로를 /로 이동한 다음
root 폴더의 소유권을 사용자 소유권으로 바꾼다. (-R 옵션은 하위경로들도 권한을 바꾼다)
그 후에 fabric-samples/asset-transfer-basic/chaincode-javascript 경로로 이동한다.
code . 을 사용해 vscode 실행

code . 은 경로의 vscode를 실행하는 명령어로, 사용하려면 vscode에 Reomte-WSL이 설치되어 있어야 한다.

``` 
cd /
chown -R ljwoon /root
cd /fabric-samples/asset-transfer-basic/chaincode-javascript
code .
```

![image](https://user-images.githubusercontent.com/68358404/121765067-4db82480-cb83-11eb-8b5e-65655b7715ef.png)

lib폴더에 assetTransfer를 보면
/asset-transfer-basic/chaincode-javascript 안에 있는 체인코드가 무엇을 하는지 알 수 있다.
스마트 계약으로 가져오고 자산 전송 클래스를 만드는 데 사용되는 계약 클래스를 볼 수 있다.

![image](https://user-images.githubusercontent.com/68358404/121765342-27938400-cb85-11eb-83a3-d01553ffabe1.png)

체인코드를 패키징 할 수 있도록 모듈설치

```
npm install
```

![image](https://user-images.githubusercontent.com/68358404/121765224-5a894800-cb84-11eb-88ce-4ce107fd707d.png)

다시 test-network로 이동

```
cd ../../test-network
```

peerCLI를 사용하여 체인 코드 패키지를 생성 할 수 있다. peerCLI를 사용할 수 있게 환경변수 설정.

```
su root
export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=$PWD/../config/
peer version
```

![image](https://user-images.githubusercontent.com/68358404/121765764-54956600-cb88-11eb-8c6b-29b36f642f7f.png)

체인코드 패키지 생성

```
peer lifecycle chaincode package basic.tar.gz --path ../asset-transfer-basic/chaincode-javascript/ --lang node --label basic_1.0
```

다음 명령어는 현재 디렉토리에 ../asset-transfer-basic/chaincode-javascript/ 경로에 있는 노드 언어인 체인코드를 basic.tar.gz로 패키징 
--label은 체인코드를 실행할때 사용되는 id 
체인코드를 패키징 완료하면 네트워크의 피어에게 설치할 수 있다.


## 3-5. 체인코드 패키지 설치

 체인 코드는 트랜잭션을 승인 할 모든 피어에 설치되어야합니다. Org1과 Org2 모두의 보증을 요구하도록 보증 정책을 설정할 것이므로 두 조직에서 운영하는 피어에 체인 코드를 설치

 우선 Org1 피어에 체인 코드를 설치하기위해선 peerCLI를 사용. peerCLI를 사용하기 위해 환경변수 설정.
 
```
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```

![image](https://user-images.githubusercontent.com/68358404/121765776-742c8e80-cb88-11eb-8153-e7ea42350ada.png)

피어 라이프 사이클 체인 코드 설치 명령을 실행 하여 피어에 체인 코드 를 설치.

```
peer lifecycle chaincode install basic.tar.gz
```

![image](https://user-images.githubusercontent.com/68358404/121765822-90c8c680-cb88-11eb-98fe-fef1dd10ff73.png)

이제 Org2 피어에 체인 코드를 설치. 환경변수 설정

```
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
```

![image](https://user-images.githubusercontent.com/68358404/121765847-c077ce80-cb88-11eb-8316-65a6c371c9fa.png)

```
peer lifecycle chaincode install basic.tar.gz
```
![image](https://user-images.githubusercontent.com/68358404/121765864-f1580380-cb88-11eb-95b1-73d889c6cf18.png)

네트워크에 있는 피어에게 체인코드 설치 완료

## 3-6. 체인코드 정의 승인

 체인 코드 패키지를 설치 한 후 조직한테 체인 코드 정의를 승인 받아야한다. 체인코드 패키지 ID는 피어에 설치된 체인 코드를 승인된 체인코드 정의와 연결하는 데 사용되며, 조직은 체인 코드를 사용하여 트랜잭션을 보증 할 수 있다.

체인코드 패키지 아이디를 찾는 명령어

```
peer lifecycle chaincode queryinstalled
```

![image](https://user-images.githubusercontent.com/68358404/121765976-adb1c980-cb89-11eb-86af-b841eac2989b.png)

패키지 아이디 복사
basic_1.0:d3fa7d19384042eda5a9f5d8239dd64d4f1e0ca00e6e33759ddbc86bf7840a69

체인 코드를 승인 할 때 패키지 ID를 사용할 것이므로 계속해서 환경 변수로 저장

```
export CC_PACKAGE_ID=basic_1.0:d3fa7d19384042eda5a9f5d8239dd64d4f1e0ca00e6e33759ddbc86bf7840a69
```

echo $CC_PACKAGE_ID 로 환경변수가 제대로 저장되었는지 확인 가능

체인코드 승인 명령어 

```
peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name basic --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem"
```

![image](https://user-images.githubusercontent.com/68358404/121766058-5a8c4680-cb8a-11eb-88c5-478f66e2cbab.png)

체인코드 승인

체인 코드 정의를 Org1에서도 승인 해야한다.
환경변수를 Org1으로 변경

```
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_ADDRESS=localhost:7051
```

체인 코드 정의를 Org1로 승인

```
peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name basic --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem"
```

![image](https://user-images.githubusercontent.com/68358404/121766168-1188c200-cb8b-11eb-9db3-72e692adbc21.png)

## 3-7. 체인 코드 정의를 채널에 커밋
 
 충분한 수의 조직이 체인 코드 정의를 승인하면 한 조직이 체인 코드 정의를 채널에 커밋 할 수 있다. 대다수의 채널 구성원이 정의를 승인 한 경우 커밋 트랜잭션이 성공하고 체인 코드 정의에서 동의 한 매개 변수가 채널에서 구현된다.
 peer lifecycle chaincode checkcommitreadiness 명령을 사용하여 채널 구성원이 동일한 체인 코드 정의를 승인했는지 여부를 확인할 수 있다.

![image](https://user-images.githubusercontent.com/68358404/121766208-6fb5a500-cb8b-11eb-9334-364e7ea3f89a.png)

peer lifecycle chaincode commit 명령을 사용하여 체인 코드 정의를 채널에 커밋 할 수 있다.

```
peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name basic --version 1.0 --sequence 1 --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt"
```

![image](https://user-images.githubusercontent.com/68358404/121766325-ee124700-cb8b-11eb-8664-547240835ec3.png)

peer lifecycle chaincode querycommitted 명령을 사용하여 체인 코드 정의가 채널에 커밋되었는지 확인할 수 있다.

```
peer lifecycle chaincode querycommitted --channelID mychannel --name basic --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem"
```

![image](https://user-images.githubusercontent.com/68358404/121766342-08e4bb80-cb8c-11eb-8dac-bf0c35d529dc.png)

## 3-8. 체인코드 호출

자산 전송 (기본) 체인 코드는 이제 클라이언트 애플리케이션에서 호출 할 준비가 되었다. 다음 명령을 사용하여 원장에 초기 자산 세트를 만든다.

```
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -c '{"function":"InitLedger","Args":[]}'
```

![image](https://user-images.githubusercontent.com/68358404/121766435-b0fa8480-cb8c-11eb-892e-77e1be7a32b0.png)

쿼리 함수를 사용하여 체인 코드로 생성 된 자동차 세트를 읽을 수 있다.

```
peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
```

![image](https://user-images.githubusercontent.com/68358404/121766457-d8515180-cb8c-11eb-8976-2aa3566e1357.png)

## 3-9. 스마트 계약 업데이트

동 일한 Fabric 체인 코드 라이프 사이클 프로세스를 사용하여 이미 채널에 배치 된 체인 코드를 업그레이드 할 수 있다. 채널 구성원은 새 체인 코드 패키지를 설치 한 다음 새 패키지 ID, 새 체인 코드 버전 및 1 씩 증가 된 시퀀스 번호로 체인 코드 정의를 승인하여 체인 코드를 업그레이드 할 수 있다. 새 체인 코드는 체인 코드 정의가 채널에 커밋 된 후에 사용할 수 있다. 이 프로세스를 통해 채널 구성원은 체인 코드가 업그레이드 될 때 조정하고 채널에 배포하기 전에 충분한 수의 채널 구성원이 새 체인 코드를 사용할 준비가되었는지 확인할 수 있다.

```
export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=$PWD/../config/
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
peer lifecycle chaincode package basic_2.tar.gz --path ../asset-transfer-basic/chaincode-javascript/ --lang node --label basic_2.0
```

체인코드를 2번째로 피키징 하기에 이름과 label을 2.0으로 지음
peerCLI를 Org1으로 환경변수 설정

```
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```

체인코드 설치

```
peer lifecycle chaincode install basic_2.tar.gz
```

체인코드 패키지 아이디 확인

```
peer lifecycle chaincode queryinstalled
```

basic_2.0:80febd83fde021becfaacba6c8b3241ebc4bf65b8a8d146ed7679699c990465e

체인코드 패키지 아이디 환경변수에 등록
echo $NEW_CC_PACKAGE_ID 로 확인 가능

```
export NEW_CC_PACKAGE_ID=basic_2.0:80febd83fde021becfaacba6c8b3241ebc4bf65b8a8d146ed7679699c990465e
```

Org1은 이제 새 체인 코드 정의를 승인 할 수 있다.

```
peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name basic --version 2.0 --package-id $NEW_CC_PACKAGE_ID --sequence 2 --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem"
```

마찬가지로 Org2에 체인코드 패키지를 설치하고 체인코드 정의

```
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
```

체인코드 패키지 설치

```
peer lifecycle chaincode install basic_2.tar.gz
```

체인코드 채널 승인

```
peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name basic --version 2.0 --package-id $NEW_CC_PACKAGE_ID --sequence 2 --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem"
```

peer lifecycle chaincode checkcommitreadiness 명령을 사용하여 시퀀스 2의 체인 코드 정의가 채널에 커밋 될 준비가되었는지 확인

```
peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name basic --version 2.0 --sequence 2 --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" --output json
```

체인 코드는 새 체인 코드 정의가 커밋 된 후 채널에서 업그레이드된다. 그때까지 이전 체인 코드는 두 조직의 피어에서 계속 실행됩니다. Org2는 다음 명령을 사용하여 체인 코드를 업그레이드 할 수 있다.

```
peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name basic --version 2.0 --sequence 2 --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt"
```

새 체인 코드가 피어에서 시작되었는지 확인할 수 있다  docker ps

```
docker ps
```

![image](https://user-images.githubusercontent.com/68358404/121767091-308a5280-cb91-11eb-8c64-e1382075b22f.png)


새 체인코드 테스트 

```
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -c '{"function":"CreateAsset","Args":["asset8","blue","16","Kelley","750"]}'
```

![image](https://user-images.githubusercontent.com/68358404/121767105-40a23200-cb91-11eb-8d24-ee4566acc32a.png)


원장의 모든 자동차를 다시 쿼리하여 새 자동차를 볼 수 있다.

```
peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
```

![image](https://user-images.githubusercontent.com/68358404/121767113-4c8df400-cb91-11eb-8419-a82979429a78.png)

## 3-10. 네트워크 종료

logspout 종료

```
docker stop logspout
docker rm logspout
```

네트워크 종료

```
./network.sh down
```
------------------------------------------------------------

## 4-1. 패브릭 애플리케이션 실행

 이 튜토리얼은 Fabric 애플리케이션이 배치 된 블록 체인 네트워크와 상호 작용하는 방법에 대한 소개한다. Fabric SDK를 사용하여 빌드된 샘플 프로그램을 사용하여 스마트 계약 처리에 자세히 설명된 스마트 계약 API로 원장을 쿼리하고 업데이트하는 스마트 계약을 호출한다. 그리고 샘플 프로그램과 배포된 인증기관을 사용하여 애플리케이션이 허가된 블록 체인과 상호 작용하는데 필요한 X.509 인증서를 생성한다.
 
## 4-2. 네트워크 실행 

네트워크 종료, 사용하지 않는 도커 메시지 삭제, 네트워크 실행 myhannel 이름의 채널 생성 CA인증서도 같이 실행

```
./network.sh down
docker network prune
./network.sh up createChannel -c mychannel -ca
```

![image](https://user-images.githubusercontent.com/68358404/121768259-4d765400-cb98-11eb-8943-1df2b1b037fe.png)


 이미 승인된 체인코드를 채널에 배포

```
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-javascript/ -ccl javascript
 ```
 
 ![image](https://user-images.githubusercontent.com/68358404/121768293-86aec400-cb98-11eb-8dc9-8ee2d9ff9d97.png)

## 4-3. 샘플 애플리케이션 실행

새 터미널 열고

```
cd fabric-samples/asset-transfer-basic/application-javascript
npm install
node app.js
```

![image](https://user-images.githubusercontent.com/68358404/121770195-ead68580-cba2-11eb-8d13-74b060592b9d.png)

완료된 모습 error는 잘못된게 아니다. 원래 나오는 부분

![image](https://user-images.githubusercontent.com/68358404/121770273-5fa9bf80-cba3-11eb-8494-0588e71538b7.png)

경로에 code . 입력해 app.js 파일을 분석한다.
enrollAdmin 은 wallet을 만들고 거기에 인증기관(CA)에서 관리자 자격 증명이 담긴다. wallet/admin.id 파일 에서 관리자의 인증서와 개인 키를 찾을 수 있다.

![image](https://user-images.githubusercontent.com/68358404/121770311-94b61200-cba3-11eb-9fae-f6b0052bcc02.png)

registerAndEnrollUser은 블록체인 네트워크와 상호 작용하는데 사용할 앱 사용자의 id를 등록(regist)하고 인증서와 개인키 등록(enroll)

![image](https://user-images.githubusercontent.com/68358404/121770488-86b4c100-cba4-11eb-8d23-1f20617841f2.png)

 애플리케이션이 게이트웨이를 통해 계약 이름과 채널 이름을 사용하여 계약에 대한 참조를 받고 있음을 알 수 있다.
 
```
contract = await network.getContract('chaincodeName', 'smartContractName');
```
위 코드는 체인 코드 패키지에 여러 스마트 계약이 포함 된 경우 getContract () API 에서 체인 코드 패키지의 이름과 대상으로 지정할 특정 스마트 계약을 모두 지정할 수 있다.

![image](https://user-images.githubusercontent.com/68358404/121770543-d8f5e200-cba4-11eb-9fe3-fe324bd16ad6.png)

submitTransaction () 함수는 InitLedger일부 샘플 데이터로 원장을 채우기 위해 chaincode 함수를 호출하는데 사용된다 . 내부적으로 submitTransaction () 함수는 서비스 검색을 사용하여 체인 코드에 필요한 승인 피어 세트를 찾고, 필요한 피어 수에서 체인 코드를 호출하고, 해당 피어에서 체인 코드 승인 결과를 수집하고, 마지막으로 orderer 노드에게 트랜잭션을 제출한다. 

![image](https://user-images.githubusercontent.com/68358404/121770587-165a6f80-cba5-11eb-9e49-2dfea4337119.png)

체인코드 호출

![image](https://user-images.githubusercontent.com/68358404/121770605-2eca8a00-cba5-11eb-866e-73058fd27c0f.png)

호출 결과

![image](https://user-images.githubusercontent.com/68358404/121770654-8668f580-cba5-11eb-9815-c6837b1e4a52.png)

이 에러 부분은 체인코드 'UpdateAsset'은 존재하지 않는 자산(asset70)에 대한 트랜잭션 제출을 시도하다 나온 것이다.

## 4-4. 그외에 코드 분석

```
const { Gateway, Wallets } = require('fabric-network');
```

위 코드는 Wallat에서 appUser의 신원을 찾고 네트워크를 연결하는데 사용

```
await gateway.connect(ccp, {
  wallet,
  identity: userId,
  discovery: {enabled: true, asLocalhost: true} 
});
```

지갑에 저장된 사용자 ID로 게이트웨이 연결을 설정하고 검색 옵션을 지정함

```
const ccpPath = path.resolve(__dirname, '..', '..', 'test-network','organizations','peerOrganizations','org1.example.com', 'connection-org1.json');
```

ccpPath 변수는 애플리케이션 네트워크에 연결하기 위해 사용되는 접속 프로파일에 대한 경로를 설명

```
await contract.submitTransaction('InitLedger');
```

다양한 트랜잭션이 있으며 InitLedger 트랜잭션을 사용해 초기화. world state (현재 상태)를 채움

## 4-5. 네트워크 종료

```
./network.sh down
```

## 5-1. 상업 어음 튜토리얼

이 튜토리얼은 상업 어음 샘플 애플리케이션과 스마트 계약을 설치하고 사용하는 방법을 보여준다.

## 5-2. 시나리오

![image](https://user-images.githubusercontent.com/68358404/121771088-2162cf00-cba8-11eb-92b3-e8345913d603.png)

조직 : MagnetoCorp, DigiBank 
개발자 : MagnetoCorpDevelop 
관리자 : MagnetoCorpAdmin, DigiBankAdmin 
사용자 : Isabella(MagnetoCorp소속), Balaji(DigiBank) 

Isabella가 어음 발행 
Balaji가 어음 구매, 상환

## 5-3. 네트워크 실행

![image](https://user-images.githubusercontent.com/68358404/121771205-f0cf6500-cba8-11eb-9b91-9c30b1cfa16c.png)

2개의 peer 노드 과 하나의 orderer 노드 각자 CA 존재. 
Org1을 DigiBank로, Org2를 MagnetoCorp로 운영

```
cd fabric-samples/commercial-paper
./network-starter.sh
```

