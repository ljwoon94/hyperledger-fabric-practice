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

WSL2 설치 (Ubuntu 20.04.2 LTS)
Docker Desktop 최신버전
go, jq, curl ,git, nvm 을 우분투에다 설치

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
## 테스트 네트워크 실행
 
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


------------------------------------------

## 채널 만들기

 채널은 특정 네트워크 구성원 간의 개인 통신 계층이다. 채널은 채널에 초대 된 조직에서만 사용할 수 있으며 네트워크의 다른 구성원에게는 표시되지 않습다. 각 채널에는 별도의 블록 체인 원장이 있다. 초대를받은 조직은 피어를 채널에 "참여"하여 채널 원장을 저장하고 채널에서 트랜잭션을 검증한다.
 
 ./network.sh를 사용해 Org1 과 Org2 사이의 채널 만들기.
 
 createChannel -c <채널명>

```
./network.sh createChannel -c mychannel
```

![image](https://user-images.githubusercontent.com/68358404/121638874-a5dd2100-cac6-11eb-966e-67f30ec0cd2b.png)

-----------------------------------------

## 채널에서 체인코드 시작

 채널을 만든 후 스마트 계약 을 사용하여 채널 원장과 상호 작용할 수 있다. 스마트 계약(체인코드)는 블록 체인 원장의 자산을 관리하는 비즈니스 로직이 포함되어 있다. 네트워크 구성원이 실행하는 애플리케이션은 스마트 계약(체인코드)을 호출하여 원장에 자산을 생성하고 해당 자산을 변경 및 전송할 수 있다. 애플리케이션은 원장의 데이터를 읽기 위해 스마트 계약을 쿼리한다.
 
 Fabric에서 스마트 계약은 체인 코드라고하는 패키지로 네트워크에 배포된다. 체인 코드는 조직의 피어에 설치되고 채널에 배포되어 트랜잭션을 승인하고 블록 체인 원장과 상호 작용하는 데 사용할 수 있다. 체인 코드를 채널에 배포하려면 채널 구성원이 체인 코드 거버넌스를 설정하는 체인 코드 정의에 동의해야한다. 필요한 수의 조직이 동의하면 체인 코드 정의를 채널에 커밋 할 수 있으며 체인 코드를 사용할 준비가 된 것이다.
 
 이미 승인된 체인코드를 채널에 배포 해보기
 deployCC 체인코드 배포하는 명령어, -ccn 체인코드 네임, -ccp 체인코드 경로 , -ccl 체인코드 언어
 채널을 지정하지 않으면 기본 mychannel로 지정된다.
 
```
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
```

--------------------------------------

## 네트워크와 상호작용

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

## 네트워크 중단

```
./network.sh down
```
