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
$ curl -sSL https://bit.ly/2ysbOFE | bash -s -- 2.2.3 1.5.0
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

혹시 실행 했는데 다음과 같은 오류가 나온다면 </br></br>
https://kudpassion.com/%ED%95%98%EC%9D%B4%ED%8D%BC-%EB%A0%88%EC%A0%80-js-1/ </br></br>
<img height="auto" src="https://user-images.githubusercontent.com/54825978/121109063-cd718680-c845-11eb-81cc-a057a74b7afc.png">
</br>
위 에러는 네트워크의 설정이 잘못 되어 발생 되는 것을 보아 </br>
◾ docker-compose.yaml </br>
◾ core.yaml 이 문제가 있다고 판단
</br></br>
◾ docker-compose-test-net.yaml에 있는 </br>
CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_test  를 모두 </br>
CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=docker_test로 변경
<img height="auto" src="https://user-images.githubusercontent.com/54825978/121109355-5b4d7180-c846-11eb-8c51-79ad28f929fb.png">
<img height="auto" src="https://user-images.githubusercontent.com/54825978/121109411-78824000-c846-11eb-8777-496fecdba63e.png">
</br>
test-network/config </br>

◾ core.yaml에 있는 NetworkMode: host </br>

NetworkMode: docker_test로 변경

<img height="auto" src="https://user-images.githubusercontent.com/54825978/121109659-f5151e80-c846-11eb-8b41-c7fb734670a4.png">
</br>

◾ 네트워크 재실행 후 처음부터 다시한다.

-------------------

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
docker_test 네트워크 로그 감시

```
cp ../commercial-paper/organization/digibank/configuration/cli/monitordocker.sh .
./monitordocker.sh docker_test
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

![image](https://user-images.githubusercontent.com/68358404/121978438-28b1f480-cdc3-11eb-9946-9ae73addf66b.png)

패키지 아이디 복사
basic_1.0:56c7ab51ebdb0530b36ee9ca0e624e2ae01807dc8793fe4475e2efc98bc7bd14

체인 코드를 승인 할 때 패키지 ID를 사용할 것이므로 계속해서 환경 변수로 저장

```
export CC_PACKAGE_ID=basic_1.0:56c7ab51ebdb0530b36ee9ca0e624e2ae01807dc8793fe4475e2efc98bc7bd14
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

----------------------------------------------------

## 5-1. 상업 어음 튜토리얼

이 튜토리얼은 상업 어음 샘플 애플리케이션과 스마트 계약을 설치하고 사용하는 방법을 보여준다. 
Fabric SDK는 트랜잭션 승인, 주문 및 알림 프로세스를 처리하여 애플리케이션의 논리를 간단하게 만든다. SDK는 게이트웨이 를 사용하여 네트워크 세부 정보와 connectionOptions 를 추상화 하여 트랜잭션 재시도와 같은 고급 처리 전략을 선언한다.

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

2개의 peer 노드 과 하나의 orderer 노드 각자 CA 존재. couchdb 2개 있다.
Org1을 DigiBank로, Org2를 MagnetoCorp로 운영

```
cd fabric-samples/commercial-paper
./network-starter.sh
```

![image](https://user-images.githubusercontent.com/68358404/121792583-f1600e00-cc31-11eb-8d8a-cd5c11c4975b.png)

## 5-4. MagnetoCorp로 네트워크 모니터링

MagnetoCorp Admin은 현재 PaperNet의 구성 요소를 모니터링 한다.

```
cd commercial-paper/organization/magnetocorp
./configuration/cli/monitordocker.sh docker_test
```

![image](https://user-images.githubusercontent.com/68358404/121792806-561c6800-cc34-11eb-8d63-6213e5e1a0b9.png)

## 5-5. 상업 어음 스마트 계약 검토

상업 어음 스마트 계약의 핵심기능은 3가지가 있다. issue(어음 발행), buy(어음 구매), redeem(어음 상환)
commercial-paper/organization/magnetocorp 경로로 가 contract/lib/papercontract.js 코드를 확인해본다.
wsl2에서 vscode를 실행하려면 root권한이 아닌 상태로 경로로 가 code. 을 입력한다.
권한땜에 접속하기 힘드면 chown -R 사용자계정 /경로 를 입력한다.

```
cd commercial-paper/organization/magnetocorp
code .
```

![image](https://user-images.githubusercontent.com/68358404/121792884-389bce00-cc35-11eb-9693-366af245fca2.png)

class CommercialPaperContract extends Contract {
CommercialPaperContract 클래스는 내장된 Fabric Contract클래스를 기반으로 스마트 계약 클래스를 정의

## 5-6. 채널에 스마트 계약 배포
### 5-6-1. MagnetoCorp로 체인코드 설치 및 승인

MagnetoCorp와 DigiBank의 admin으로서 체인 코드를 설치하고 승인해야한다.
MagnetoCorp admin은 papercontract를 MagnetoCorp피어에 복사본을 설치한다.
MagnetoCorp admin이 네트워크와 상호작용하려면 peerCLI 사용해야한다.
샘플에서 제공하는 스크립트를 사용하여 명령 창에서 peerCLI를 사용하기위한 환경 변수를 설정할 수 있습니다.

```
cd commercial-paper/organization/magnetocorp
source magnetocorp.sh
```

환경변수 설정완료

![image](https://user-images.githubusercontent.com/68358404/121793039-4a31a580-cc36-11eb-8966-b429def504f6.png)

체인코드 패키징

```
peer lifecycle chaincode package cp.tar.gz --lang node --path ./contract --label cp_0
```

MagnetoCorp peer에 체인코드 설치

```
peer lifecycle chaincode install cp.tar.gz
```

![image](https://user-images.githubusercontent.com/68358404/121793075-9846a900-cc36-11eb-8031-70df0e9551c7.png)

조직(MagnetoCorp)에게 체인코드 정의를 승인이 필요한다. 승인을 하기 위핸 package_id가 필요하다.

```
peer lifecycle chaincode queryinstalled
```

체인코드 Package_id

![image](https://user-images.githubusercontent.com/68358404/121982170-fd7ed380-cdc9-11eb-853a-2c480deb3ddc.png)

Package_id 환경변수에 등록 echo $PACKAGE_ID로 등록됐는지 확인 가능

```
export PACKAGE_ID=cp_0:ddca913c004eb34f36dfb0b4c0bcc6d4afc1fa823520bb5966a3bfcf1808f40a
```

체인코드 승인

```
peer lifecycle chaincode approveformyorg --orderer localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name papercontract -v 0 --package-id $PACKAGE_ID --sequence 1 --tls --cafile $ORDERER_CA
```

![image](https://user-images.githubusercontent.com/68358404/121793213-dd1f0f80-cc37-11eb-9253-f2a9bf0749d9.png)

### 5-6-2. DigiBank로 스마트 계약 설치 및 승인

DigiBank admin이 체인코드 설치 및 승인
네트워크와 상호작용 하기위해 필요한 peerCLI를 사용하기 위해 환경변수가 적혀있는 스크립트 실행  

```
cd commercial-paper/organization/digibank/
source digibank.sh
```

![image](https://user-images.githubusercontent.com/68358404/121793238-18214300-cc38-11eb-887b-d10cb6525976.png)

체인코드 패키징

```
peer lifecycle chaincode package cp.tar.gz --lang node --path ./contract --label cp_0
```

![image](https://user-images.githubusercontent.com/68358404/121793269-74846280-cc38-11eb-9402-470338884666.png)

체인코드 설치

```
peer lifecycle chaincode install cp.tar.gz
```

조직에 체인코드 승인하기위해 Package_id 등록

```
peer lifecycle chaincode queryinstalled
```

```
export PACKAGE_ID=cp_0:ddca913c004eb34f36dfb0b4c0bcc6d4afc1fa823520bb5966a3bfcf1808f40a
```

조직에 체인코드 승인

```
peer lifecycle chaincode approveformyorg --orderer localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name papercontract -v 0 --package-id $PACKAGE_ID --sequence 1 --tls --cafile $ORDERER_CA
```

![image](https://user-images.githubusercontent.com/68358404/121793652-40ab3c00-cc3c-11eb-88ea-63485a59a4d4.png)

## 5-7. 체인 코드 정의를 채널에 커밋

체인코드가 채널에서 성공적으로 정의되면 채널의 클라이언트 애플리케이션에서 체인코드 CommercialPaper를 호출할 수 있다. DigiBank admin은 코드의 정의를 papercontract채널에 커밋한다.

```
peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --peerAddresses localhost:7051 --tlsRootCertFiles ${PEER0_ORG1_CA} --peerAddresses localhost:9051 --tlsRootCertFiles ${PEER0_ORG2_CA} --channelID mychannel --name papercontract -v 0 --sequence 1 --tls --cafile $ORDERER_CA --waitForEvent
```

![image](https://user-images.githubusercontent.com/68358404/121794544-0a71ba80-cc44-11eb-96f8-c37a87c62c2e.png)

## 5-8. 신청 구조

MagnetoCorp의 Isabella는 어플리케이션에서 체인코드(issue)를 사용하여 상업 어음을 발행한다.

![image](https://user-images.githubusercontent.com/68358404/121794647-0e520c80-cc45-11eb-9ade-365769b5e41b.png)

어플리케이션에서 요청하면 gateway가 피어에게 트랜잭션을 제출한다. 피어는 트랜잭션이 맞는지 확인후 SWset에 담아서 다시 gateway로
보낸다. 그리고 gateway는 orderer에 값을 보내 Ordere는 블록을 생성하고, 다른 피어들에게 트랜잭션을 보낸다.

MagnetoCorp의 isabella가 사용하는 기능

```
cd commercial-paper/organization/magnetocorp/application/
code issue.js
```

지갑에서 id 사용해 본인 식별 
const wallet = await Wallets.newFileSystemWallet('../identity/user/isabella/wallet');


issue.js 에 gateway를 사용해 네트워크에 접속
await gateway.connect(connectionProfile, connectionOptions);

![image](https://user-images.githubusercontent.com/68358404/121794903-32aee880-cc47-11eb-8c58-b922eb529987.png)


const contract = await network.getContract('papercontract');

const issueResponse = await contract.submitTransaction('issue', 'MagnetoCorp', '00001', ...);
submitTransaction을 사용해 스타트 계약 트랜젝션을 제출할 수 있다.

## 5-9. 지갑

Isabell의 신분 확인, 인증서 발급.

```
npm install
node enrollUser.js
```

wallet이라는 경로에 인증서와, 비밀키가 생성된다.

## 5-10. 발급 신청

이제 issue.js 를 실행해 상업 어음을 발행할 수 있다.

```
node issue.js
```

![image](https://user-images.githubusercontent.com/68358404/121798857-a5c65800-cc63-11eb-9d7f-d29e7d5ffeab.png)

## 5-11. Digibank 애플리케이션

 DigiBank의 체인코드(buy)을 사용 하여 거래를 원장에 제출하여 00001MagnetoCorp에서 DigiBank로 상업 어음 소유권을 이전한다.
 
```
cd commercial-paper/organization/digibank/application/
code buy.js
```
buy.js 코드분석

const buyResponse = await contract.submitTransaction('buy', 'MagnetoCorp', '00001', ...);
buy 트랜잭션 호출

사용자 등록

```
cd commercial-paper/organization/digibank/application/
npm install
node enrollUser.js
```

![image](https://user-images.githubusercontent.com/68358404/121799098-04d89c80-cc65-11eb-9d58-b0a2f08c2093.png)

어음 구매

```
node buy.js
```

![image](https://user-images.githubusercontent.com/68358404/121799131-2fc2f080-cc65-11eb-9c9f-f963f08c01d6.png)

어음 상환

node redeem.js

![image](https://user-images.githubusercontent.com/68358404/121799158-639e1600-cc65-11eb-8930-832035a72c35.png)

## 5-12. 네트워크 종료
```
cd fabric-samples/commercial-paper
./network-clean.sh
```

--------------------------------------------------

## 6-1. Fabric에서 개인 데이터 사용

 이 튜토리얼은 개인 데이터 수집(PDC)을 사용하여 인증된 조직 동료를 위해 블록체인 네트워크에서 개인 데이터의 저장 및 검색을 제공하는 방법을 보여준다. 컬렉션은 해당 컬렉션을 관리하는 정책이 포함된 컬렉션 정의 파일을 사용하여 지정된다.
 
## 6-2. 자산 전송 개인 데이터 샘플 사용 사례

 이 샘플은 Org1과 Org2 간에 자산을 전송하기 위해 세 개(assetCollection, Org1MSPPrivateCollection, Org2MSPPrivateCollection)의 개인 데이터 수집 및 사용을 보여준다.
 Org1의 구성원은 이제 소유자라고하는 새 자산을 작성한다. 소유자의 신원을 포함한 자산의 공개 세부 정보는라는 비공개 데이터 컬렉션(assetCollection)에 저장된다. 
 자산은 또한 소유자가 제공한 평가 가치로 생성된다. 평가된 가치는 각 참가자가 자산 양도에 동의하는데 사용되며 소유자 조직의 컬렉션(Org1MSPPrivateCollection)에만 저장된다. 
 자산을 구매하려면 구매자가 자산 소유자와 동일한 평가 가치에 동의 해야한다. 이 단계에서 구매자(Org2의 구성원)는 거래 계약을 작성하고 스마트 계약 기능(AgreeToTransfer)을 사용하여 평가 가치에 동의한다. 이 값은 Org2MSPPrivateCollection컬렉션에 저장된다 . 이제 자산 소유자는 스마트 계약 기능을 사용하여 자산을 구매자에게 양도 할 수 있다.
 'TransferAsset'함수는 채널 원장의 해시를 사용하여 자산을 이전하기 전에 소유자와 구매자가 동일한 평가값에 동의했는지 확인한다.
  
 ## 6-3. 컬렉션 정의 JSON 파일 빌드
 
 일련의 조직이 개인 데이터를 사용하여 거래하기 전에 채널의 모든 조직은 각 체인 코드와 연관된 개인 데이터 콜렉션을 정의하는 콜렉션 정의 파일을 빌드해야한다. 개인 데이터 컬렉션에 저장된 데이터는 채널의 모든 구성원이 아닌 특정 조직의 피어에게만 배포된다. 컬렉션 정의 파일은 조직이 체인 코드에서 읽고 쓸 수있는 모든 개인 데이터 컬렉션을 설명한다.각 컬렉션은 다음 속성으로 정의된다.
 
* name: 컬렉션의 이름이다.
* policy: 컬렉션 데이터를 유지할 수있는 조직 피어를 정의한다.
* requiredPeerCount: 체인 코드 보증 조건으로 개인 정보 유포에 필요한 피어의 수
* maxPeerCount: 데이터 중복성을 위해 현재 보증하는 피어가 데이터 배포를 시도 할 다른 피어의 수이다. 보증 피어가 다운되면 개인 데이터를 가져 오라는 요청이있는 경우 커밋시 이러한 다른 피어를 사용할 수 있다.
* blockToLive: 가격 또는 개인 정보와 같이 매우 민감한 정보의 경우이 값은 데이터가 블록 측면에서 개인 데이터베이스에 얼마나 오래 있어야 하는지를 나타낸다. 데이터는 개인 데이터베이스에서이 지정된 수의 블록 동안 유지되며 그 후에 제거되어이 데이터가 네트워크에서 쓸모 없게됩니다. 개인 데이터를 무기한 유지하려면, 즉 개인 데이터를 절대 제거하지 않으려면 blockToLive속성을로 0으로 설정한다 .
* memberOnlyRead: 값은 true피어가 컬렉션 구성원 조직 중 하나에 속한 클라이언트 만 개인 데이터에 대한 읽기 액세스를 허용하도록 자동으로 적용 함을 나타낸다.
* memberOnlyWrite: 값은 true피어가 컬렉션 구성원 조직 중 하나에 속한 클라이언트 만 개인 데이터에 대한 쓰기 액세스를 허용하도록 자동으로 적용 함을 나타낸다.
* endorsementPolicy: 개인 데이터 수집에 쓰기 위해 충족해야하는 보증 정책을 정의한다. 컬렉션 수준 보증 정책은 체인 코드 수준 정책으로 재정의된다. 정책 정의 작성에 대한 자세한 정보는 보증 정책 주제를 참조한다.

조직이 어떤 컬렉션에도 속하지 않더라도 체인 코드를 사용하는 모든 조직에서 동일한 컬렉션 정의 파일을 배포해야한다. 
자산 전송 개인 데이터 예제에는 세 개의 개인 정의 하는 collections_config.json 파일이 포함되어 있다.
fabric_samples/asset-transfer-private-data/chaincode-go/ 경로에 있다.

![image](https://user-images.githubusercontent.com/68358404/121829078-7f083000-ccfc-11eb-9351-f6f1f4b82d17.png)


assetCollection 정의한 policy속성은 Org1과 Org2가 모두 해당 피어에 컬렉션을 저장할 수 있음을 지정한다. 
"memberOnlyRead": true, "memberOnlyWrite": true member가 컬랙션을 읽거나 쓸 수 있다. 

이 컬렉션 정의 파일은 peer lifecycle chaincode commit 명령을 사용하여 체인 코드 정의가 채널에 커밋 될 때 배포된다.

## 6-4. 체인 코드 API를 사용하여 개인 데이터 읽기 및 쓰기

 자산 전송 개인 데이터 샘플은 데이터에 액세스하는 방법에 따라 개인 데이터를 세 개의 개별 데이터 정의로 나눈다.
 
 ![image](https://user-images.githubusercontent.com/68358404/121839695-9acc0000-cd15-11eb-85ee-9f40c4442f83.png)

assetCollection에 objectType, color, size, owner이 저장된다
수집 정책 (Org1 및 Org2)의 정의에 따라 상기 채널의 부재로 표시한다.

자산 전송 개인 데이터 샘플 스마트 계약에 의해 생성 된 모든 데이터는 PDC에 저장된다.

collections_config.json 는 Org1 및 Org2의 모든 피어가 개인 데이터베이스에 자산 전송 개인 데이터를 저장하고 거래 할 수 있도록 허용
하지만 Org1 컬랙션에 있는 appraisedValue는 Org1.member만 저장하고 거래할 수 있다. Org2 컬랙션에 있는 appraisedValue도 Org2.member만 저장하고 거래가능하다.

## 6-5. 네트워크 시작, 개인 데이터 스마트 계약을 채널에 배포

네트워크 실행

```
cd fabric-samples/test-network
./network.sh down
./network.sh up createChannel -ca -s couchdb
```

개인 데이터 스마트 계약을 채널에 배포

```
./network.sh deployCC -ccn private -ccp ../asset-transfer-private-data/chaincode-go/ -ccl go -ccep "OR('Org1MSP.peer','Org2MSP.peer')" -cccg ../asset-transfer-private-data/chaincode-go/collections_config.json
```

## 6-6. 신원등록

 개인 데이터 전송 스마트 계약은 네트워크에 속한 개별 ID의 소유권을 지원한다. 이 시나리오에서 자산 소유자는 Org1의 구성원이되고 구매자는 Org2에 속한다. GetClientIdentity().GetID()API와 사용자 인증서 내의 정보 간의 연결을 강조하기 위해 Org1 및 Org2 인증 기관 (CA)을 사용하여 두 개의 새 ID를 등록한 다음 CA를 사용하여 각 ID의 인증서와 개인 키를 생성한다.
 
Fabric CA 클라이언트를 사용하가위해 다음 환경 변수를 설정

```
export PATH=${PWD}/../bin:${PWD}:$PATH
export FABRIC_CFG_PATH=$PWD/../config/
```

Org1 CA를 사용하여 ID 자산 소유자를 만듭니다. Fabric CA 클라이언트 홈을 Org1 CA 관리자의 MSP로 설정

```
export FABRIC_CA_CLIENT_HOME=${PWD}/organizations/peerOrganizations/org1.example.com/
```

fabric-ca-client 도구를 사용하여 새 소유자 클라이언트 ID를 등록 할 수 있다.

```
fabric-ca-client register --caname ca-org1 --id.name owner --id.secret ownerpw --id.type client --tls.certfiles "${PWD}/organizations/fabric-ca/org1/tls-cert.pem"
```

이제 등록 명령에 등록 이름과 암호를 제공하여 ID 인증서와 MSP 폴더를 생성 할 수 있다.

```
fabric-ca-client enroll -u https://owner:ownerpw@localhost:7054 --caname ca-org1 -M "${PWD}/organizations/peerOrganizations/org1.example.com/users/owner@org1.example.com/msp" --tls.certfiles "${PWD}/organizations/fabric-ca/org1/tls-cert.pem"
```

노드 OU 구성 파일을 소유자 ID MSP 폴더에 복사한다.

```
cp "${PWD}/organizations/peerOrganizations/org1.example.com/msp/config.yaml" "${PWD}/organizations/peerOrganizations/org1.example.com/users/owner@org1.example.com/msp/config.yaml"\
```

Org2 CA를 사용하여 구매자 ID를 만들 수 있습니다. Fabric CA 클라이언트 홈을 Org2 CA 관리자로 설정

```
export FABRIC_CA_CLIENT_HOME=${PWD}/organizations/peerOrganizations/org2.example.com/
```
fabric-ca-client 도구를 사용하여 새 소유자 클라이언트 ID를 등록 

```
fabric-ca-client register --caname ca-org2 --id.name buyer --id.secret buyerpw --id.type client --tls.certfiles "${PWD}/organizations/fabric-ca/org2/tls-cert.pem"
```

이제 등록하여 ID MSP 폴더를 생성 

```
fabric-ca-client enroll -u https://buyer:buyerpw@localhost:8054 --caname ca-org2 -M "${PWD}/organizations/peerOrganizations/org2.example.com/users/buyer@org2.example.com/msp" --tls.certfiles "${PWD}/organizations/fabric-ca/org2/tls-cert.pem"
```

아래 명령을 실행하여 노드 OU 구성 파일을 구매자 ID MSP 폴더에 복사

```
cp "${PWD}/organizations/peerOrganizations/org2.example.com/msp/config.yaml" "${PWD}/organizations/peerOrganizations/org2.example.com/users/buyer@org2.example.com/msp/config.yaml"
```

## 6-7. 개인 데이터에 자산 만들기

소유자의 신원을 생성 했으므로 개인 데이터 스마트 계약을 호출하여 새 자산을 생성 할 수 있다. test-network 디렉토리의 터미널에 환경변수 설정.

```
export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=$PWD/../config/
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/owner@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```

CreateAsset함수를 사용하여 아이디 asset1, 색상 green, 크기 20, appraisedValue 100인 개인 데이터에 저장된 자산을 생성

```
export ASSET_PROPERTIES=$(echo -n "{\"objectType\":\"asset\",\"assetID\":\"asset1\",\"color\":\"green\",\"size\":20,\"appraisedValue\":100}" | base64 | tr -d \\n)
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n private -c '{"function":"CreateAsset","Args":[]}' --transient "{\"asset_properties\":\"$ASSET_PROPERTIES\"}"
```

결과

![image](https://user-images.githubusercontent.com/68358404/121849595-7b899e80-cd26-11eb-891c-159ce56d5fe6.png)

ReadAsset 함수를 사용하여 assetCollection 컬렉션을 Org1 로 쿼리 하여 생성 된 자산의 주요 세부 정보를 읽기

```
peer chaincode query -C mychannel -n private -c '{"function":"ReadAsset","Args":["asset1"]}'
```

결과

![image](https://user-images.githubusercontent.com/68358404/121850343-97417480-cd27-11eb-88d5-e15096d8ff21.png)

이제 Org1의 구성원으로 개인 데이터에 저장된 asset1의 appraisedValue를 쿼리 

```
peer chaincode query -C mychannel -n private -c '{"function":"ReadAssetPrivateDetails","Args":["Org1MSPPrivateCollection","asset1"]}'
```

결과

![image](https://user-images.githubusercontent.com/68358404/121850561-ebe4ef80-cd27-11eb-95c1-94fd1fbcdefe.png)

## 6-7. 권한이없는 피어로 개인 데이터 쿼리

Org2의 피어로 전환

```
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/buyer@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
```

Org2의 피어는 사이드 데이터베이스에 asset1 호출

```
peer chaincode query -C mychannel -n private -c '{"function":"ReadAsset","Args":["asset1"]}'
```

결과

![image](https://user-images.githubusercontent.com/68358404/121850791-41b99780-cd28-11eb-8806-abca9d1a0551.png)


ReadAssetPrivateDetails 함수에 asset1을 호출 시도
Org2 컬랙션에는 저장이 된 것이 없어 아무것도 실행되지 않는다.

```
peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n private -c '{"function":"ReadAssetPrivateDetails","Args":["Org2MSPPrivateCollection","asset1"]}'
```

결과 한줄 띄어져 있다.

![image](https://user-images.githubusercontent.com/68358404/121851143-bab8ef00-cd28-11eb-8994-57051160bca1.png)

Org2의 사용자는 Org1 개인 데이터 콜렉션을 읽을 수도 없다.

```
peer chaincode query -C mychannel -n private -c '{"function":"ReadAssetPrivateDetails","Args":["Org1MSPPrivateCollection","asset1"]}'
```

결과

![image](https://user-images.githubusercontent.com/68358404/121851253-de7c3500-cd28-11eb-82c9-f94ad0674fe8.png)

## 6-8. 자산 양도

asset1을 Org2로 이전 시켜본다. appraisedValue 을 옮기려면 자산을 구입하는데 동의가 필요하다.

자산을 이전하려면 구매자 (수신자)가 appraisedValuechaincode 함수를 호출하여 자산 소유자 와 동일하게 동의해야한다.
합의 된 값은 Org2MSPDetailsCollectionOrg2 피어 의 컬렉션에 저장

```
export ASSET_VALUE=$(echo -n "{\"assetID\":\"asset1\",\"appraisedValue\":100}" | base64 | tr -d \\n)
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n private -c '{"function":"AgreeToTransfer","Args":[]}' --transient "{\"asset_value\":\"$ASSET_VALUE\"}"
```

![image](https://user-images.githubusercontent.com/68358404/121986005-c5c75a00-cdd0-11eb-9498-d1eefef9db77.png)

구매자는 이제 Org2 개인 데이터 수집에서 동의한 값을 쿼리 가능

```
peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n private -c '{"function":"ReadAssetPrivateDetails","Args":["Org2MSPPrivateCollection","asset1"]}'
```

![image](https://user-images.githubusercontent.com/68358404/121986033-d4157600-cdd0-11eb-933f-bf03d23f2fcc.png)

구매자가 평가 된 가치를 위해 자산을 구매하는 데 동의 했으므로 소유자는 자산을 Org2로 이전 할 수 있다. 자산은 자산을 소유 한 ID로 전송해야하므로 Org1으로 작동한다.

```
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/owner@org1.example.com/msp
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_ADDRESS=localhost:7051
```

Org1의 소유자는 AgreeToTransfer 트랜잭션에서 추가 한 데이터를 읽고 구매자 ID를 볼 수 있다.

```
peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n private -c '{"function":"ReadTransferAgreement","Args":["asset1"]}'
```

![image](https://user-images.githubusercontent.com/68358404/121986157-03c47e00-cdd1-11eb-996d-0c4d00cd4a10.png)

자산을 전송하려면 다음 명령을 실행, 소유자는 이전 거래에 구매자의 assetID 및 조직 MSP ID를 제공 해야한다.

```
export ASSET_OWNER=$(echo -n "{\"assetID\":\"asset1\",\"buyerMSP\":\"Org2MSP\"}" | base64 | tr -d \\n)
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n private -c '{"function":"TransferAsset","Args":[]}' --transient "{\"asset_owner\":\"$ASSET_OWNER\"}" --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```

결과 이제 구매자 ID가 자산을 소유하고 있음을 보여준다.

![image](https://user-images.githubusercontent.com/68358404/121986284-2ce50e80-cdd1-11eb-8211-a4d0e6ca67a7.png)


트랜잭션이 Org1 컬렉션에서 비공개 세부 정보를 삭제했는지 확인가능

```
peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n private -c '{"function":"ReadAssetPrivateDetails","Args":["Org1MSPPrivateCollection","asset1"]}'
```

결과 

![image](https://user-images.githubusercontent.com/68358404/121986510-949b5980-cdd1-11eb-9e16-b237bf7d286b.png)

## 6-9. 개인 데이터 삭제

 Org2MSPPrivateCollection 정의는 blockToLive 의 속성 값에 의해 3개 블록을 데이터베이스에 거주하고 블록이 추가되면 이전 데이터가 삭제된다. 채널에 추가 블록을 생성하면 appraisedValueOrg2가 동의 한 블록이 결국 제거됩니다. 시연하기 위해 3 개의 새 블록을 만는다.
 
Org2로 변경

```
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/buyer@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051 
```

값 확인

```
peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n private -c '{"function":"ReadAssetPrivateDetails","Args":["Org2MSPPrivateCollection","asset1"]}'
```

![image](https://user-images.githubusercontent.com/68358404/121987142-d2e54880-cdd2-11eb-972f-349e4aac48ae.png)

개인 데이터가 데거되기 전에 추가하는 블록 수를 추적하기 위해 새 터미널을 열고 데이터 로그 확인

```
docker logs peer0.org1.example.com 2>&1 | grep -i -a -E 'private|pvt|privdata'
```

Org2의 구성원으로 작동하는 터미널로 돌아가 블록 3개 생성

```
export ASSET_PROPERTIES=$(echo -n "{\"objectType\":\"asset\",\"assetID\":\"asset2\",\"color\":\"blue\",\"size\":30,\"appraisedValue\":100}" | base64 | tr -d \\n)
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n private -c '{"function":"CreateAsset","Args":[]}' --transient "{\"asset_properties\":\"$ASSET_PROPERTIES\"}"
```
```
export ASSET_PROPERTIES=$(echo -n "{\"objectType\":\"asset\",\"assetID\":\"asset3\",\"color\":\"red\",\"size\":25,\"appraisedValue\":100}" | base64 | tr -d \\n)
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n private -c '{"function":"CreateAsset","Args":[]}' --transient "{\"asset_properties\":\"$ASSET_PROPERTIES\"}"
```
```
export ASSET_PROPERTIES=$(echo -n "{\"objectType\":\"asset\",\"assetID\":\"asset4\",\"color\":\"orange\",\"size\":15,\"appraisedValue\":100}" | base64 | tr -d \\n)
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n private -c '{"function":"CreateAsset","Args":[]}' --transient "{\"asset_properties\":\"$ASSET_PROPERTIES\"}"
```

![image](https://user-images.githubusercontent.com/68358404/121987346-471fec00-cdd3-11eb-89a0-76dcf1f32006.png)

블록생성 확인

```
docker logs peer0.org1.example.com 2>&1 | grep -i -a -E 'private|pvt|privdata'
```
![image](https://user-images.githubusercontent.com/68358404/121987451-76365d80-cdd3-11eb-8f53-02a7dfedcaf3.png)

삭제됐는지 확인

```
peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n private -c '{"function":"ReadAssetPrivateDetails","Args":["Org2MSPPrivateCollection","asset1"]}'
```

![image](https://user-images.githubusercontent.com/68358404/121987491-88b09700-cdd3-11eb-922f-8389d8cb053e.png)

## 6-10. 네트워크 종료

```
./network.sh down
```

## 7-1. Fabric의 안전한 자산 전송

 튜토리얼은 Hyperledger Fabric 블록 체인 채널에서 조직간에 자산을 표현하고 거래하는 방법을 보여 주며, 개인 데이터를 사용하여 자산 및 트랜잭션의 세부 정보를 비공개로 유지한다. 소유자가 자산을 매각하려는 경우 자산을 양도하기 전에 양 당사자가 동일한 가격에 동의해야한다. 개인 자산 전송 스마트 계약은 자산 소유자 만 자산을 전송할 수 있도록 강제한다.
 
## 7-2. 시나리오 요구 사항

* 자산은 첫 번째 소유자의 조직에서 발행 할 수 있습니다 (실제 세계에서 발행은 자산의 자산을 인증하는 일부 기관으로 제한 될 수 있음).

* 소유권은 조직 수준에서 관리됩니다 (Fabric 권한 부여 체계는 조직 내의 개별 ID 수준에서 소유권을 동일하게 지원합니다).

* 자산 식별자 및 소유자는 모든 채널 구성원이 볼 수 있도록 공개 채널 데이터로 저장됩니다.

* 그러나 자산 메타 데이터 속성은 자산 소유자 (및 이전 소유자)에게만 알려진 개인 정보입니다.

* 관심있는 구매자는 자산의 사유 재산을 확인하려고합니다.

* 관심있는 구매자는 자산의 출처, 특히 자산의 출처 및 보관 체인을 확인하려고합니다. 또한 발행 이후 자산이 변경되지 않았는지, 그리고 이전의 모  든 양도가 합법적 이었는지 확인하려고합니다.

* 자산을 양도하려면 구매자와 판매자가 먼저 판매 가격에 동의해야합니다.

* 현재 소유자 만 자산을 다른 조직으로 이전 할 수 있습니다.

* 실제 개인 자산 양도는 합법적 인 자산이 양도되고 있는지 확인하고 가격에 동의했는지 확인해야합니다. 구매자와 판매자 모두 양도를 승인해야합니다.
 
## 7-3. 프라이버시가 유지되는 방법

 스마트 계약은 다음 기술을 사용하여 자산 속성이 비공개로 유지한다.

* 자산 메타 데이터 속성은 조직의 피어에서만 현재 소유 조직의 암 적 개인 데이터 컬렉션에 저장된다. Fabric 채널의 각 조직에는 자체 조직에서 사용할 수있는 개인 데이터 콜렉션이 있습니다. 이 컬렉션은 체인 코드에서 명시 적으로 정의할 필요가 없기 때문에 암시적이다.

* 개인 속성의 해시는 모든 채널 구성원이 볼 수 있도록 체인에 자동으로 저장되지만, 다른 채널 구성원이 사전 공격을 통해 개인 데이터 사전 이미지를 추측 할 수 없도록 개인 속성에 무작위 솔트가 포함됩니다.

* 스마트 계약 요청은 개인 데이터에 대해 임시 필드를 활용하므로 개인 데이터가 최종 온 체인 트랜잭션에 포함되지 않습니다.

* 비공개 데이터 쿼리는 조직 ID가 피어의 조직 ID와 일치하는 클라이언트에서 시작되어야하며 자산 소유자의 조직 ID와 동일해야합니다.
 
## 7-4. 전송이 구현되는 방법
 
### 7-4-1. 자산 생성
 
 개인 자산 전송 스마트 계약은 모든 채널 구성원의 보증을 요구하는 보증 정책과 함께 배포된다. 이를 통해 모든 조직은 다른 채널 구성원의 승인없이 소유 한 자산을 만들 수 있다. 자산 생성은 체인 코드 수준 보증 정책을 사용하는 유일한 트랜잭션이다. 기존 자산을 업데이트하거나 이전하는 거래는 주 기반 보증 정책 또는 개인 데이터 수집의 보증 정책에 따라 관리된다. 다른 시나리오에서는 발행 기관이 트랜잭션 작성을 보증하도록 할 수도 있다.
 
### 7-4-2. 양도에 동의

 자산이 생성 된 후 채널 구성원은 스마트 계약을 사용하여 자산 전송에 동의 할 수 있다.
 저작물의 소유자는 예를 들어 저작물이 판매 중임을 광고하기 위해 공개 소유권 기록의 설명을 변경할 수 있습니다. 스마트 계약 액세스 제어는 자산 소유자 조직의 구성원이이 변경 사항을 제출해야합니다. 주 기반 보증 정책은이 설명 변경이 소유자 조직의 동료에 의해 승인되어야한다.
 

### 7-4-1. 자산 양도
 
 두 조직이 동일한 가격에 동의하면 자산 소유자는 이전 기능을 사용하여 자산을 구매자에게 이전 할 수 있다.
 
## 7-5. 개인 자산 이전 스마트 계약 실행

 Fabric 테스트 네트워크를 사용하여 개인 자산 전송 스마트 계약을 실행할 수 있다. 테스트 네트워크에는 각각 하나의 피어를 운영하는 두 개의 피어 조직 Org1 및 Org2가 포함된다. 이 튜토리얼에서는 두 조직이 결합한 테스트 네트워크의 채널에 스마트 계약을 배포한다. 먼저 Org1이 소유 한 자산을 생성한다. 두 조직이 가격에 동의하면 자산을 Org1에서 Org2로 이전한다.

## 7-6. 테스트 네트워크 배포

```
cd fabric-samples/test-network
./network.sh down
./network.sh up createChannel -c mychannel
```

## 7-7. 테스트 네트워크 배포

테스트 네트워크 스크립트를 사용하여 보안 자산 전송 스마트 계약을 채널에 배포

```
./network.sh deployCC -ccn secured -ccp ../asset-transfer-secured-agreement/chaincode-go/ -ccl go -ccep "OR('Org1MSP.peer','Org2MSP.peer')"
```

## 7-8. Org1와 Org2로 작동하도록 환경 변수 2개 설정

Org1

```
export PATH=${PWD}/../bin:${PWD}:$PATH
export FABRIC_CFG_PATH=$PWD/../config/
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_ADDRESS=localhost:7051
```

새 터미널 열고 test-network에서 Org2 환경변수 설정

```
export PATH=${PWD}/../bin:${PWD}:$PATH
export FABRIC_CFG_PATH=$PWD/../config/
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_ADDRESS=localhost:9051
```

## 7-9. 자산 생성

 모든 채널 구성원은 스마트 계약을 사용하여 조직이 소유 한 자산을 만들 수 있다. 자산의 세부 정보는 개인 데이터 컬렉션에 저장되며 자산을 소유 한 조직 만 액세스 할 수 있다. 자산, 소유자 및 공개 설명에 대한 공개 기록은 채널 원장에 저장된다. 모든 채널 회원은 공개 소유권 기록에 액세스하여 저작물 소유자를 확인할 수 있으며 설명을 읽고 저작물이 판매 중인지 확인할 수 있다.
 
### 7-9-1. Org1 터미널에서 작동

 자산을 생성하기 전에 자산의 세부 사항을 지정해야한다. 다음 명령을 실행하여 자산을 설명하는 JSON을 생성한다. "salt"매개 변수는 원장의 해시를 사용하여 자산을 추측에서 채널의 다른 회원을 방해하는 임의의 문자열이다. 솔트가없는 경우 사용자는 추측의 해시와 원장의 해시가 일치 할 때까지 이론적으로 자산 매개 변수를 추측 할 수 있다 

```
export ASSET_PROPERTIES=$(echo -n "{\"object_type\":\"asset_properties\",\"asset_id\":\"asset1\",\"color\":\"blue\",\"size\":35,\"salt\":\"a94a8fe5ccb19ba61c4c0873d391e987982fbbd3\"}" | base64 | tr -d \\n)
```

Org1에 속하는 자산 생성

```
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"CreateAsset","Args":["asset1", "A new asset for Org1MSP"]}' --transient "{\"asset_properties\":\"$ASSET_PROPERTIES\"}"
```

![image](https://user-images.githubusercontent.com/68358404/121997482-632c8900-cde5-11eb-95e2-1b4af9b17700.png)

Org1 생성된 자산 확인

```
peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"GetAssetPrivateProperties","Args":["asset1"]}'
```

![image](https://user-images.githubusercontent.com/68358404/121997570-8820fc00-cde5-11eb-9af1-14f8527a8c48.png)

```
peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"ReadAsset","Args":["asset1"]}'
```

원장을 쿼리하여 공용 소유권을 볼 수 있다.

```
peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"ReadAsset","Args":["asset1"]}'
```

![image](https://user-images.githubusercontent.com/68358404/121997653-a981e800-cde5-11eb-8a88-67b08513bb94.png)

 Org1은이 자산을 매각하려고한다. 자산 소유자로서 Org1은 자산이 판매중임을 알리기 위해 공개 설명을 업데이트 할 수 있습니다. 자산 설명을 변경하려면 다음 명령을 실행한다.
 
 ```
 peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"ChangePublicDescription","Args":["asset1","This asset is for sale"]}'
 ```
 
 ![image](https://user-images.githubusercontent.com/68358404/121997760-d209e200-cde5-11eb-9e71-72928e065fe1.png)
 
 원장을 다시 물어 업데이트 된 설명을 확인
 
 ```
 peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"ReadAsset","Args":["asset1"]}'
 ```
 
![image](https://user-images.githubusercontent.com/68358404/121997823-ecdc5680-cde5-11eb-9e20-62e5e9903c4c.png)

### 7-9-2. Org2 터미널에서 작동

Org2 터미널에서 작동하는 경우 스마트 계약을 사용하여 공용 자산 데이터를 쿼리 할 수 있다.

```
peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"ReadAsset","Args":["asset1"]}'
```

이 쿼리에서 Org2는 asset1이 판매 중임을 알게된다.

![image](https://user-images.githubusercontent.com/68358404/121998070-42b0fe80-cde6-11eb-830e-33b8b3a6302c.png)

Org2가 공개 설명을 장난으로 변경하려고하면 어떻게되는지 본다.

```
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"ChangePublicDescription","Args":["asset1","the worst asset"]}'
```

스마트 계약은 Org2가 자산의 공개 설명에 액세스하는 것을 허용하지 않는다.

![image](https://user-images.githubusercontent.com/68358404/121998180-755af700-cde6-11eb-9018-0ca3022e1c5d.png)

## 7-10. 자산 매각에 동의

 자산을 판매하려면 구매자와 판매자 모두 자산 가격에 동의해야한다. 각 당사자는 자신의 개인 데이터 수집에 동의 한 가격을 저장한다. 개인 자산 이전 스마트 계약은 자산을 이전하기 전에 양 당사자가 동일한 가격에 동의해야한다고 강제한다.

## 7-10-1. Org1으로 판매하기로 동의

 Org1 터미널에서 작동한다. Org1은 자산 가격을 110 달러로 설정하는 데 동의한다. trade_id는 구매자 또는 가격을 추측에서 판매되지 않는 채널 회원을 방지하기 위해 소금으로 사용된다. 이 값은 구매자와 판매자간에 이메일 또는 기타 통신을 통해 대역 외로 전달되어야한다. 구매자와 판매자는 채널의 다른 구성원이 판매 할 자산을 추측하지 못하도록 자산 키에 소금을 추가 할 수도 있다.
 
 ```
 export ASSET_PRICE=$(echo -n "{\"asset_id\":\"asset1\",\"trade_id\":\"109f4b3c50d7b0df729d299bc6f8e9ef9066971f\",\"price\":110}" | base64)
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"AgreeToSell","Args":["asset1"]}' --transient "{\"asset_price\":\"$ASSET_PRICE\"}"
 ```
