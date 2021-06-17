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
cp "${PWD}/organizations/peerOrganizations/org1.example.com/msp/config.yaml" "${PWD}/organizations/peerOrganizations/org1.example.com/users/owner@org1.example.com/msp/config.yaml"
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

------------------------------------------------

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

Org2가 공개 설명을 장난으로 변경하려고하면 어떻게 되는지 본다.

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
export ASSET_PRICE=$(echo -n "{\"asset_id\":\"asset1\",\"trade_id\":\"109f4b3c50d7b0df729d299bc6f8e9ef9066971f\",\"price\":110}" | base64 | tr -d \\n)
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"AgreeToSell","Args":["asset1"]}' --transient "{\"asset_price\":\"$ASSET_PRICE\"}"
```

![image](https://user-images.githubusercontent.com/68358404/122001591-82c6b000-cdeb-11eb-8191-7eb76b613b05.png)


```
peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"GetAssetBidPrice","Args":["asset1"]}'
```

![image](https://user-images.githubusercontent.com/68358404/122001637-9540e980-cdeb-11eb-9522-8b2b0f27c181.png)

## 7-10-2. Org2으로 구매하는데 동의

Org2 터미널에서 작동한다. 구매에 동의하기 전에 다음 명령을 실행하여 자산 속성을 확인. 자산 속성과 소금은 구매자와 판매자간에 이메일 또는 기타 통신을 통해 대역 외로 전달된다.

```
export ASSET_PROPERTIES=$(echo -n "{\"object_type\":\"asset_properties\",\"asset_id\":\"asset1\",\"color\":\"blue\",\"size\":35,\"salt\":\"a94a8fe5ccb19ba61c4c0873d391e987982fbbd3\"}" | base64 | tr -d \\n)
peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"VerifyAssetProperties","Args":["asset1"]}' --transient "{\"asset_properties\":\"$ASSET_PROPERTIES\"}"
```

![image](https://user-images.githubusercontent.com/68358404/122001814-d6d19480-cdeb-11eb-8581-5786a83f62cf.png)

 다음 명령을 실행하여 asset1을 100 달러에 구매하는 데 동의한다. 현재 Org2는 Org2와 다른 가격에 동의한다. 두 조직은 향후 단계에서 동일한 가격에 동의 할 것이다. 그러나 우리는이 일시적인 불일치를 구매자와 판매자가 다른 가격에 동의하면 어떻게되는지 테스트 할 수 있다. Org2는 Org1과  trade_id 동일하게 사용 해야한다.
 
```
export ASSET_PRICE=$(echo -n "{\"asset_id\":\"asset1\",\"trade_id\":\"109f4b3c50d7b0df729d299bc6f8e9ef9066971f\",\"price\":100}" | base64 | tr -d \\n)
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"AgreeToBuy","Args":["asset1"]}' --transient "{\"asset_price\":\"$ASSET_PRICE\"}"
```

![image](https://user-images.githubusercontent.com/68358404/122002827-34b2ac00-cded-11eb-924c-f5ab31a01608.png)


Org2에서 합의 된 구매 가격을 읽을 수 있습니다.

```
peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"GetAssetBidPrice","Args":["asset1"]}'
```

![image](https://user-images.githubusercontent.com/68358404/122002956-5f046980-cded-11eb-9feb-299bece71990.png)

## 7-11. Org1에서 Org2로 자산 이전

 두 조직이 가격에 동의 한 후 Org1은 자산을 Org2로 이전 할 수 있다. 스마트 계약의 개인 자산 이전 기능은 원장의 해시를 사용하여 두 조직이 동일한 가격에 동의했는지 확인한다. 이 함수는 또한 개인 자산 세부 정보의 해시를 사용하여 전송 된 자산이 Org1이 소유 한 자산과 동일한 지 확인한다.
 
### 7-11-1. 자산을 Org1로 이전

 Org1 터미널에서 작동한다. 자산 소유자가 이전을 시작 해야한다. 아래 명령어는 --peerAddresses플래그를 사용하여 Org1 및 Org2의 피어를 대상으로 한다. 두 조직 모두 이전을 승인해야한다. 또한 자산 속성 및 가격은 전송 요청에서 임시 속성으로 전달된다. 이는 현재 소유자가 올바른 자산이 올바른 가격으로 이전되었는지 확인할 수 있도록 전달된다. 이러한 속성은 두 보증인이 온 체인 해시에 대해 확인한다.

```
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"TransferAsset","Args":["asset1","Org2MSP"]}' --transient "{\"asset_properties\":\"$ASSET_PROPERTIES\",\"asset_price\":\"$ASSET_PRICE\"}" --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
```

두 조직이 동일한 가격에 동의하지 않았기 때문에 이전을 완료 할 수 없다.

![image](https://user-images.githubusercontent.com/68358404/122003416-05e90580-cdee-11eb-8364-ec8cf4129fe6.png)

결과적으로 Org1과 Org2는 자산을 구매할 가격에 대한 새로운 계약을 맺는다. Org1은 자산 가격을 100으로 떨어 뜨린다.

```
export ASSET_PRICE=$(echo -n "{\"asset_id\":\"asset1\",\"trade_id\":\"109f4b3c50d7b0df729d299bc6f8e9ef9066971f\",\"price\":100}" | base64 | tr -d \\n)
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"AgreeToSell","Args":["asset1"]}' --transient "{\"asset_price\":\"$ASSET_PRICE\"}"
```

![image](https://user-images.githubusercontent.com/68358404/122003639-506a8200-cdee-11eb-94d1-48b263de13d6.png)

구매자와 판매자가 동일한 가격에 동의 했으므로 Org1은 자산을 Org2로 이전 할 수 있다.

```
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"TransferAsset","Args":["asset1","Org2MSP"]}' --transient "{\"asset_properties\":\"$ASSET_PROPERTIES\",\"asset_price\":\"$ASSET_PRICE\"}" --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
```

![image](https://user-images.githubusercontent.com/68358404/122003706-6415e880-cdee-11eb-930c-8424dd72e5e8.png)

저작물 소유권 레코드를 쿼리하여 이전이 성공했는지 확인할 수 있다.

```
peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"ReadAsset","Args":["asset1"]}'
```

이제 레코드에 Org2가 자산 소유자로 나열된다.

![image](https://user-images.githubusercontent.com/68358404/122003821-8b6cb580-cdee-11eb-833e-f99cd67814d6.png)

### 7-11-2. 자산 설명을 Org2로 업데이트

 Org2 터미널에서 작동한다. 이제 Org2가 자산을 소유하고 있으므로 Org2에서 자산 세부 정보를 읽을 수 있다.
 
 ```
 peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"GetAssetPrivateProperties","Args":["asset1"]}'
 ```

![image](https://user-images.githubusercontent.com/68358404/122004031-d5559b80-cdee-11eb-90d9-0c81d10e612f.png)

Org2는 이제 자산 공개 설명을 업데이트 할 수 있다.

```
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"ChangePublicDescription","Args":["asset1","This asset is not for sale"]}'
```

![image](https://user-images.githubusercontent.com/68358404/122004432-5f9dff80-cdef-11eb-91b9-1bbdfb0f630e.png)

원장을 질의하여 자산이 더 이상 판매되지 않는지 확인

![image](https://user-images.githubusercontent.com/68358404/122004499-73e1fc80-cdef-11eb-9aa4-64a3a34d1d03.png)

## 7-12. 네트워크 종료

./network.sh down

----------------------------------------------

## 8-1. CouchDB

 하이퍼레저 패브릭은 CouchDB를 사용한다. Docker로 사용할 수 있으며, 피어와 동일한 서버에서 실행하는 것이 좋다.
 CouchDB 켄테이너는 Core.yaml에서 설정해 피어 당 하나의 CouchDB를 할당하고, 각 피어 컨테이너를 업데이트 해야한다.
 
## 8-2. 인덱스 만들기

 mongoDB랑 유사하다. 

- 예제

 ![image](https://user-images.githubusercontent.com/68358404/122141369-1bab0900-ce88-11eb-9d4e-a790b4e28a95.png)

인덱스를 만들고 난 후 체인코드가 있는 META-INF/statedb/couchdb/indexes 경로에 넣어 함께 패키징하고 설치할 수 있다.

## 8-3. 네트워크 실행

```
cd ../asset-transfer-ledger-queries/chaincode-go
GO111MODULE=on go mod vendor
cd ../../test-network
./network.sh up createChannel -s couchdb
```

## 8-4. 스마트 계약 배포

mychannel에 스마트 계약 배포
-ccep 플래그를 사용하여 “OR ( 'Org1MSP.peer', 'Org2MSP.peer')” 보증 정책으로 스마트 계약을 배포하고 있다. 이를 통해 어느 한 조직이 다른 조직의 보증을받지 않고도 자산을 만들 수 있다.

```
./network.sh deployCC -ccn ledger -ccp ../asset-transfer-ledger-queries/chaincode-go/ -ccl go -ccep "OR('Org1MSP.peer','Org2MSP.peer')"
```

## 8-5. 인덱스 배포되었는지 확인

 체인 코드가 피어에 설치되고 채널에 배포되면 인덱스가 각 피어의 CouchDB 상태 데이터베이스에 배포된다. Docker 컨테이너에서 피어 로그를 검사하여 CouchDB 인덱스가 성공적으로 생성되었는지 확인할 수 있다.
 
 피어 Docker 컨테이너의 로그를 보려면 새 터미널 창을 열고 다음 명령을 실행하여 인덱스가 생성되었다는 메시지 확인을 위해 grep한다.
 
```
docker logs peer0.org1.example.com 2>&1 | grep "CouchDB index"
```

![image](https://user-images.githubusercontent.com/68358404/122141973-69744100-ce89-11eb-95ce-21943bbd567d.png)

## 8-6. CouchDB 상태 데이터베이스 쿼리

 인덱스가 JSON 파일에 정의되고 체인 코드와 함께 배포되었으므로 체인 코드 함수는 CouchDB 상태 데이터베이스에 대해 JSON 쿼리를 실행할 수 있다. 쿼리에 인덱스 이름을 지정하는 것은 선택사항이다. 지정하지 않고 쿼리중인 필드에 대한 색인이 이미 존재하는 경우 기존 색인이 자동으로 사용된다.

-Tip use_index 키워드를 사용하여 쿼리에 인덱스 이름을 명시적으로 포함하는 것이 좋다. 그것이 있으면 CouchDB는 최적의 인덱스를 선택할 수 있다. 또한 CouchDB는 인덱스를 전혀 사용하지 않을 수 있으며 테스트하는 동안 낮은 볼륨에서이를 인식하지 못할 수도 있다. 더 높은 볼륨에서만 CouchDB가 인덱스를 사용하지 않기 때문에 성능이 저하 될 수 있다.


## 8-7. 체인 코드로 쿼리 작성

 체인 코드 내에 정의 된 쿼리를 사용하여 원장의 데이터에 대해 JSON 쿼리를 수행 할 수 있다. 자산 전송 원장 쿼리 샘플에는 2개의 JSON 쿼리 기능을 포함한다.
 
* QueryAssets

 임시 JSON 쿼리. JSON 쿼리 문자열을 함수에 전달할 수있는 쿼리이다. 이 쿼리는 런타임에 고유 한 쿼리를 동적으로 빌드해야하는 클라이언트 응용 프로그램에 유용하다.

* QueryAssetsByOwner

 매개 변수화 된 쿼리, 쿼리에 전달되는 쿼리 매개 변수를 chaincode에 정의할 수 있다. 함수가 단일 인수, 자산 소유자를 받아이 경우. 그런 다음 상태 데이터베이스에서 "asset"의 docType 및 JSON 쿼리 구문을 사용하여 소유자 ID와 일치하는 JSON 문서를 쿼리한다.

## 8-8. peer 명령을 사용하여 쿼리 실행

 클라이언트 애플리케이션이없는 경우 peer 명령을 사용하여 체인 코드에 정의된 쿼리를 테스트 할 수 있다. 우리는 사용 피어 chaincode 쿼리 자산 인덱스(indexOwner)를 사용하는 명령 QueryAssets을 사용. 
 
 
데이터베이스를 쿼리하기 전에 데이터를 추가해야합니다. 다음 명령을 Org1로 실행하여 "tom"이 소유 한 자산을 작성 해야한다.

```
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n ledger -c '{"Args":["CreateAsset","asset1","blue","5","tom","35"]}'
```

![image](https://user-images.githubusercontent.com/68358404/122149269-c4f8fb80-ce96-11eb-8b26-bb9cbeed9d09.png)

tom이 소유한 모든 자산을 띄우기.

```
peer chaincode query -C mychannel -n ledger -c '{"Args":["QueryAssets", "{\"selector\":{\"docType\":\"asset\",\"owner\":\"tom\"}, \"use_index\":[\"_design/indexOwnerDoc\", \"indexOwner\"]}"]}'
```

위 쿼리 명령을 보면 세가지 인수가 있다.

* QueryAssets

 자산 체인 코드에있는 함수의 이름입니다. 아래 chaincode 함수에서 볼 수 있듯이 QueryAssets()는 getQueryResultForQueryString()호출 한 다음 getQueryResult() 상태 데이터베이스에 대해 JSON 쿼리를 실행하는 shim API에 queryString을 전달한다.

* {"selector":{"docType":"asset","owner":"tom"}

* "use_index":["_design/indexOwnerDoc", "indexOwner"]

 디자인 문서 이름(indexOwnerDoc)과 인덱스 이름(indexOwner)을 모두 지정한다. selector 쿼리는 use_index 를 써서 인덱스 이름을 포한한다.
디자인 문서 이름은 8-2. 인덱스 만들기 과정에서 들어간 "ddoc":"indexOwnerDoc" 을 사용한다. 이 둘을 사용해야 최적화가 된다.

결과

![image](https://user-images.githubusercontent.com/68358404/122149290-cf1afa00-ce96-11eb-81c9-ee4a883279d8.png)

## 8-8. 쿼리 및 인덱스에 대한 모범 사례 사용

 이 섹션의 예는 쿼리에서 인덱스를 사용하는 방법과 최상의 성능을 제공하는 쿼리 유형을 보여준다. 쿼리를 작성할 때 다음 사항을 기억해자.
 
* 인덱스의 모든 필드는 인덱스를 사용할 쿼리의 selector 또는 sort sections 사용해야 한다.

* 복잡한 쿼리는 성능이 떨어지고 인덱스를 사용할 가능성이 적습니다.

* 전체 테이블 스캔하는 것, 또는 or, in, regex 같은 전체 인덱스 스캔을 발생시키는 연산자를 피해야한다.

이전 섹션에서 자산 체인 코드에 대해 다음 쿼리를 실행했다.

```
// 전체 데이터베이스를 검색하지 않는 쿼리
export CHANNEL_NAME=mychannel
peer chaincode query -C $CHANNEL_NAME -n ledger -c '{"Args":["QueryAssets", "{\"selector\":{\"docType\":\"asset\",\"owner\":\"tom\"}, \"use_index\":[\"indexOwnerDoc\", \"indexOwner\"]}"]}'
```

```
// 필드를 추가해도 최적화를 유지한다.
peer chaincode query -C $CHANNEL_NAME -n ledger -c '{"Args":["QueryAssets", "{\"selector\":{\"docType\":\"asset\",\"owner\":\"tom\",\"color\":\"blue\"}, \"use_index\":[\"/indexOwnerDoc\", \"indexOwner\"]}"]}'
```

## 8-9. 페이징처리를 사용해  CouchDB 상태 데이터베이스 쿼리

 QueryAssetsWithPagination을 사용하여 체인 코드 및 클라이언트 애플리케이션에서 페이징 처리를 구현
 
 데이터 4개 추가
 
```
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile  ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n ledger -c '{"Args":["CreateAsset","asset2","yellow","5","tom","35"]}'
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile  ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n ledger -c '{"Args":["CreateAsset","asset3","green","6","tom","20"]}'
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile  ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n ledger -c '{"Args":["CreateAsset","asset4","purple","7","tom","20"]}'
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile  ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n ledger -c '{"Args":["CreateAsset","asset5","blue","8","tom","40"]}'
```

 chaincode 함수에서 볼 수 있듯이 QueryAssetsWithPagination ()은를 호출
 
![image](https://user-images.githubusercontent.com/68358404/122151508-a85ec280-ce9a-11eb-8086-3966f0e564bd.png)

체인코드 실행 5개 데이터 중 3개만 출력

```
// page size of 3:
peer chaincode query -C mychannel -n ledger -c '{"Args":["QueryAssetsWithPagination", "{\"selector\":{\"docType\":\"asset\",\"owner\":\"tom\"}, \"use_index\":[\"_design/indexOwnerDoc\", \"indexOwner\"]}","3",""]}'
```

결과

![image](https://user-images.githubusercontent.com/68358404/122159853-ef07e900-cea9-11eb-8c19-98bad7e0cb6d.png)

-Tip : 하이퍼레저 2.2버전에선 북마크 value랑 페이지 사이즈가 다르다 파라미터 변수 잘 확인하자.

다음은 pageSize를 사용하여 QueryAssetsWithPagination을 호출하는 피어 명령이다.

```
peer chaincode query -C $CHANNEL_NAME -n ledger -c '{"Args":["QueryAssetsWithPagination", "{\"selector\":{\"docType\":\"asset\",\"owner\":\"tom\"}, \"use_index\":[\"_design/indexOwnerDoc\", \"indexOwner\"]}","3","g1AAAABJeJzLYWBgYMpgSmHgKy5JLCrJTq2MT8lPzkzJBYqzJRYXp5YYg2Q5YLI5IPUgSVawJIjFXJKfm5UFANozE8s"]}'
```

결과

![image](https://user-images.githubusercontent.com/68358404/122160266-9e44c000-ceaa-11eb-8b46-810cc20704f7.png)

마지막 명령은 책갈피와 pageSize를 사용하여 QueryAssetsWithPagination을 호출하는 피어 명령
 
 ```
peer chaincode query -C $CHANNEL_NAME -n ledger -c '{"Args":["QueryAssetsWithPagination", "{\"selector\":{\"docType\":\"asset\",\"owner\":\"tom\"}, \"use_index\":[\"_design/indexOwnerDoc\", \"indexOwner\"]}","3","g1AAAABJeJzLYWBgYMpgSmHgKy5JLCrJTq2MT8lPzkzJBYqzJRYXp5aYgmQ5YLI5IPUgSVawJIjFXJKfm5UFANqBE80"]}'
 ```
 
 결과
 
![image](https://user-images.githubusercontent.com/68358404/122161412-a30a7380-ceac-11eb-83df-1a53dacdfa99.png)

## 8-10. 인덱스 업데이트

-tip: http://localhost:5984/_utils 에 접속하면 데이터 베이스 GUI 가 있다.

다음은 데이터베이스에 인덱스를 생성하는  사용할 수있는 curl 명령의 예 이다. mychannel_ledger

```
curl -i -X POST -H "Content-Type: application/json" -d"{\"index\":{\"fields\":[\"docType\",\"owner\"]},\"name\":\"indexOwner\",\"ddoc\":\"indexOwnerDoc\",\"type\":\"json\"}" http://admin:adminpw@localhost:5984/mychannel_ledger/_index
```

## 8-10. 인덱스 삭제

예제

```
 curl -X DELETE http://localhost:5984/{database_name}/_index/{design_doc}/json/{index_name} -H  "accept: */*" -H  "Host: localhost:5984"
```

튜토리얼 기준 인덱스 삭제

```
curl -X DELETE http://admin:adminpw@localhost:5984/mychannel_ledger/_index/indexOwnerDoc/json/indexOwner -H  "accept: */*" -H  "Host: localhost:5984"
```

----------------------------------------------

## 9-1 새 채널 만들기
 
 이 튜토리얼를 사용하여 configtxgen CLI 도구를 사용하여 새 채널을 생성하는 방법을 배우고 피어 채널 명령을 사용하여 피어와 채널 에 참여할 수 있다. 이 튜토리얼은 패브릭 테스트 네트워크를 활용하여 새 채널을 생성하지만이 튜토리얼의 단계는 프로덕션 환경에서 네트워크 운영자가 사용할 수도 있다.
 
## 9-2 configtxgen 도구 설정
 채널 생성 트랜잭션을 작성하고 Oderer에 트랜잭션을 제출하여 채널을 생성한다. 채널 생성 트랜잭션은 채널의 초기 구성을 지정하며 주문 서비스에서 채널 생성 블록을 작성하는 데 사용된다. 채널 생성 트랜잭션 파일을 수동으로 빌드 할 수 있지만 configtxgen 도구 를 사용하는 것이 더 쉽다. 채널 구성을 정의하는 파일(configtx.yaml)을 읽은 다음 관련 정보를 채널 생성 트랜잭션에 쓰는 방식으로 작동한다.
 
```
cd fabric-samples/test-network
export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=${PWD}/configtx
```

export PATH=${PWD}/../bin:$PATH 를 사용해, configtxgen 도구를 CLI 경로에 추가
configtxgen을 사용하려면 FABRIC_CFG_PATH환경 변수를 configtx.yaml파일 의 로컬 사본이 포함 된 디렉토리 경로로 설정

## 9-3 configtx.yaml 파일

 configtx.yaml파일은 새 채널의 채널 구성을 지정한다. 채널 구성을 빌드하는데 필요한 정보는 configtx.yaml파일 에서 읽기 및 편집 가능한 형식으로 지정된다. 이 configtxgen도구는 configtx.yaml파일에 정의 된 채널 프로필 을 사용하여 채널 구성을 만들고 Fabric에서 읽을 수있는 protobuf 형식으로 작성한다.
 
 디렉터리 configtx.yaml의 configtx폴더에서 테스트 네트워크를 배포하는데 사용되는 파일을 찾을 수 있다. 파일에는 새 채널을 만드는 데 사용할 다음 정보가 포함되어 있습니다.

Organization : 채널의 회원이 될 수있는 조직이다. 각 조직에는 채널 MSP 를 구축하는 데 사용되는 암호화 자료에 대한 참조가 있다.

Ordering Service : 어떤 주문 노드가 네트워크의 주문 서비스를 구성 할 것인지, 공통 거래 순서에 동의하는 데 사용할 합의 방법. 파일에는 주문 서비스 관리자가 될 조직도 포함된다.

채널 정책 : 파일의 여러 섹션이 함께 작동하여 조직이 채널과 상호 작용하는 방식과 채널 업데이트를 승인해야하는 조직을 제어하는 정책을 정의한다. 이 자습서에서는 Fabric에서 사용하는 기본 정책을 사용한다.

채널 프로필: 각 채널 프로필은 configtx.yaml파일 의 다른 섹션에서 가져온 정보를 참조 하여 채널 구성을 만든다. 프로파일은 주문자 시스템 채널의 생성 블록과 피어 조직에서 사용할 채널을 만드는 데 사용된다. 시스템 채널과 구별하기 위해 피어 조직에서 사용하는 채널을 종종 애플리케이션 채널이라고 한다.

이 configtxgen도구는 configtx.yaml파일을 사용하여 시스템 채널에 대한 완전한 생성 블록을 만듭니다. 따라서 시스템 채널 프로필은 전체 시스템 채널 구성을 지정해야한다. 채널 생성 트랜잭션을 만드는 데 사용되는 채널 프로필에는 응용 프로그램 채널을 만드는 데 필요한 추가 구성 정보만 있으면 된다.

## 9-4. 네트워크 시작

 configtx.yaml파일에 정의 된 두 개의 피어 조직과 단일 주문 조직으로 패브릭 네트워크를 생성 합니다. 피어 조직은 각각 하나의 피어를 운영하고 주문 서비스 관리자는 단일 주문 노드를 운영한다.
 
```
./network.sh up
```

## 9-5. 주문자 시스템 채널

 패브릭 네트워크에서 생성되는 첫 번째 채널은 시스템 채널이다. 시스템 채널은 order 서비스를 형성하는 order node 세트와 order 서비스 관리자 역할을 하는 조직 세트를 정의합니다.
 
 시스템 채널에는 블록 체인 컨소시엄의 구성원인 조직도 포함된다. 컨소시엄은 시스템 채널에 속하지만 주문 서비스의 관리자가 아닌 피어 조직의 집합이다. 컨소시엄 구성원은 새 채널을 만들고 다른 컨소시엄 조직을 채널 구성원으로 포함 할 수 있다.
 
 새로운 주문 서비스를 배포하려면 시스템 채널의 제네시스 블록이 필요하다. 테스트 네트워크 스크립트는 명령 을 실행할 때 이미 시스템 채널 생성 블록을 생성했다 . 제네시스 블록은 단일 주문 노드를 배포하는 데 사용되었으며 블록을 사용하여 시스템 채널을 만들고 네트워크의 주문 서비스를 구성했다. 
 
```
configtxgen -profile TwoOrgsOrdererGenesis -channelID system-channel -outputBlock ./system-genesis-block/genesis.block
```

configtxgen도구는 TwoOrgsOrdererGenesis 채널 프로필을 사용하여 configtx.yaml생성 블록을 작성하고 system-genesis-block폴더에 저장했다. TwoOrgsOrdererGenesis아래 프로필을 볼 수 있다.

![image](https://user-images.githubusercontent.com/68358404/122166874-2203aa00-ceb5-11eb-9063-0718f3e363d2.png)

 Orderer: 프로필의 섹션은 테스트 네트워크에서 사용하는 단일 노드 Raft 주문 서비스를 생성하면 OrdererOrg 주문 서비스 관리자입니다. Consortiums프로파일의 섹션은 이름이 피어 기관의 컨소시엄을 생성한다 SampleConsortium:. 피어 조직인 Org1과 Org2는 모두 컨소시엄의 구성원이다. 결과적으로 테스트 네트워크에서 만든 새로운 채널에 두 조직을 모두 포함 할 수 있다. 컨소시엄에 해당 조직을 추가하지 않고 다른 조직을 채널 구성원으로 추가하려는 경우 먼저 Org1 및 Org2로 채널을 만든 다음 채널 구성 을 업데이트하여 다른 조직을 추가해야 한다.

## 9-5. 응용 프로그램 채널 만들기

다음 명령을 실행하여에 대한 채널 생성 트랜잭션을 만든다.

```
configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel1.tx -channelID channel1
```

-channelID미래 채널의 이름 이 됩니다. 채널 이름은 모두 소문자이고 250 자 미만이어야하며 정규 표현식과 일치해야한다.

![image](https://user-images.githubusercontent.com/68358404/122168179-e833a300-ceb6-11eb-8c1c-3d00d513a2d8.png)

 프로필 SampleConsortium은 시스템 채널 의 이름을 참조하고 컨소시엄의 두 피어 조직을 채널 구성원으로 포함한다. 시스템 채널은 애플리케이션 채널을 생성하는 템플릿으로 사용되기 때문에 시스템 채널에 정의 된 순서 지정 노드 는 새 채널 의 기본 동의자 집합 이되고, Order 지정 서비스의 관리자는 채널의 순서 지정 관리자가됩니다. 채널 업데이트를 사용하여 동의자 집합에서 Order 노드 및 Order 조직을 추가하거나 제거 할 수 있다.

 결과
 
![image](https://user-images.githubusercontent.com/68358404/122168535-5e380a00-ceb7-11eb-907a-79c9f3765bc3.png)

peerCLI를 사용하여 채널 생성 트랜잭션을 Order 서비스에 제출할 수 있다 . peerCLI  사용하려면을 디렉토리에 FABRIC_CFG_PATH있는 core.yaml파일 로 설정해야한다.

```
export FABRIC_CFG_PATH=$PWD/../config/
```

주문 서비스가 채널을 생성하기 전에 주문 서비스는 요청을 제출 한 ID의 권한을 확인한다. 기본적으로 시스템 채널 컨소시엄에 속한 조직의 관리자 ID 만 새 채널을 만들 수 있다. peerOrg1에서 관리자로 CLI 를 작동하려면 아래 명령 을 실행한다.

```
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```

이제 다음 명령을 사용하여 채널을 만들 수 있다.
-f 를 사용하여 채널 생성 트랜잭션 파일의 경로를 제공하고 -c 를 사용하여 채널 이름을 지정한다. -o 는 채널을 생성하는데 사용되는 순서 노드를 선택하는데 사용된다. --cafile은 주문 노드의 TLS 인증서의 경로이다. 명령을 실행하면 CLI가 다음 응답을 생성한다.

```
peer channel create -o localhost:7050  --ordererTLSHostnameOverride orderer.example.com -c channel1 -f ./channel-artifacts/channel1.tx --outputBlock ./channel-artifacts/channel1.block --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

결과

![image](https://user-images.githubusercontent.com/68358404/122169155-0d74e100-ceb8-11eb-92e1-4601f1a227fd.png)

## 9-6. 채널에 동료 가입

 채널이 생성되면 동료와 함께 채널에 참여할 수 있다. 채널의 구성원 인 조직은 peer channel fetch 명령을 사용하여 순서 지정 서비스에서 채널 생성 블록을 가져올 수 있다. 그런 다음 조직은 제네시스 블록을 사용하여 peer channel join 명령을 사용하여 피어를 채널에 조인 할 수 있다. 피어가 채널에 가입되면 피어는 주문 서비스에서 채널의 다른 블록을 검색하여 블록 체인 원장을 구축한다. 
 이미 peerOrg1 관리자로 CLI를 운영하고 있으므로 Org1 피어를 채널에 가입시켜 보겠다. Org1이 채널 생성 트랜잭션을 제출했기 때문에 파일 시스템에 이미 채널 생성 블록이 있습니다. 아래 명령을 사용하여 Org1 피어를 채널에 가입시킨다.
 
```
peer channel join -b ./channel-artifacts/channel1.block
```
 
![image](https://user-images.githubusercontent.com/68358404/122170010-0b5f5200-ceb9-11eb-8547-09626e6845dd.png)

peer channel getinfo 명령을 사용하여 피어가 채널에 참여했는지 확인할 수 있다.

```
peer channel getinfo -c channel1
```

 이 명령은 채널의 블록 높이와 가장 최근 블록의 해시를 나열한다. 제네시스 블록이 채널의 유일한 블록이기 때문에 채널의 높이는 1이 된다.

![image](https://user-images.githubusercontent.com/68358404/122170186-3ea1e100-ceb9-11eb-89b4-42b7affee6f8.png)

이제 Org2 피어를 채널에 가입시킬 수 있다. peerOrg2 관리자로 CLI 를 작동하려면 다음 환경 변수를 설정한다. 환경 변수는 또한 Org2 피어를 peer0.org1.example.com대상 피어로 설정한다.

```
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
```

파일 시스템에 여전히 채널 생성 블록이 있지만보다 현실적인 시나리오에서는 Org2가 주문 서비스에서 블록을 가져 오게된다.

```
peer channel fetch 0 ./channel-artifacts/channel_org2.block -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com -c channel1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

결과

![image](https://user-images.githubusercontent.com/68358404/122170661-cc7dcc00-ceb9-11eb-81d8-516d6a49b432.png)

파일 시스템에 여전히 채널 생성 블록이 있지만보다 현실적인 시나리오에서는 Org2가 주문 서비스에서 블록을 가져 오게된다. 예를 들어, 다음 명령을 사용하여 Org2의 제네시스 블록을 가져온다.

```
peer channel join -b ./channel-artifacts/channel_org2.block
```

 명령은 0채널에 참여하는 데 필요한 제네시스 블록을 가져와야 함을 지정하는 데 사용한다. 명령이 성공하면 다음 출력이 표시되어야한다.

결과

![image](https://user-images.githubusercontent.com/68358404/122170701-d7386100-ceb9-11eb-8ee7-71885bf402e8.png)

이 명령은 채널 생성 블록을 반환하고 이름을 지정 channel_org2.block하여 org1에서 가져온 블록과 구별합니다. 이제 블록을 사용하여 Org2 피어를 채널에 연결할 수 있다.

```
![image](https://user-images.githubusercontent.com/68358404/122171143-5a59b700-ceba-11eb-823d-87b8e9177cee.png)
```

## 9-7. 앵커 피어 설정

 조직이 피어를 채널에 가입시킨 후 앵커 피어가 될 피어 중 하나 이상을 선택해야한다. 개인 데이터 및 서비스 검색과 같은 기능을 활용하려면 앵커 피어가 필요하다. 각 조직은 중복성을 위해 채널에 여러 앵커 피어를 설정해야한다.
 각 조직의 앵커 피어의 앤드포인트 정보는 채널의 구성에 포함된다. 각 채널 맴버는 채널을 업데이트 함으로써 앵커 피어를 
지정할 수 있다. 우리는 채널 구성을 업데이트하고 Org1 와 Org2를 위한 앵커피어를 선택하기 위해 configtxlator도구를 사용할 것이다. 앵커 피어를 설정하는 프로세스는 다른 채널 업데이트를 수행하는데 필요한 단계와 유사하며 채널구성을 업데이트 하는 데 사용하는 configtxlator 방법을 알려준다. 로컬 컴퓨터에 jq도구를 설치해야한다.

리눅스 jq 설치

```
curl -L https://github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64 -o /usr/local/bin/jq
chmod a+x /usr/local/bin/jq
jq -V
```

 앵커 피어를 Org1로 선택하여 시작합니다. 첫 번째 단계는 명령을 사용하여 가장 최근의 채널 구성 블록을 가져오는 것이다. CLI를 Org1 관리자로 작동하려면 다음 환경 변수를 설정한다.
 
```
export FABRIC_CFG_PATH=$PWD/../config/
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051 
```

다음 명령을 사용하여 채널 구성을 가져올 수 있다.

```
peer channel fetch config channel-artifacts/config_block.pb -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com -c channel1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

가장 최근의 채널 구성 블록은 채널 생성 블록이므로 채널에서 블록 0을 반환하는 명령이 표시

![image](https://user-images.githubusercontent.com/68358404/122313274-e0c0d800-cf50-11eb-880b-e0a0995feceb.png)

채널 구성 블록(channel-artifacts)은 다른 아티팩트와 별도로 업데이트 프로세스를 유지하기위해 폴더 에 저장되었다.

```
cd channel-artifacts
```

이제 configtxlator 도구를 사용하여 채널 구성 작업을 시작할 수 있다. 첫 번째 단계는 protobuf의 블록을 읽고 편집 할 수있는 JSON 객체로 디코딩하는 것입니다. 또한 불필요한 블록 데이터를 제거하고 채널 구성만 남긴다.

```
configtxlator proto_decode --input config_block.pb --type common.Block --output config_block.json
jq '.data.data[0].payload.data.config' config_block.json > config.json
```

 이러한 명령은 채널 구성 블록을 config.json업데이트의 기준으로 사용할 간소화 된 JSON으로 변환한다. 이 파일을 직접 편집하고 싶지 않기 때문에 편집 할 수있는 복사본을 만든다. 향후 단계에서 원래 채널 구성을 사용한다.
 
jq도구를 사용하여 Org1 앵커 피어를 채널 구성에 추가할 수 있다.

```
jq '.channel_group.groups.Application.groups.Org1MSP.values += {"AnchorPeers":{"mod_policy": "Admins","value":{"anchor_peers": [{"host": "peer0.org1.example.com","port": 7051}]},"version": "0"}}' config_copy.json > modified_config.json
```

 이 단계 후에는 modified_config.json파일 에 JSON 형식의 업데이트 된 버전의 채널 구성이 있다. 이제 원래 및 수정 된 채널 구성을 모두 다시 protobuf 형식으로 변환하고 그 차이를 계산할 수 있다.
 
```
configtxlator proto_encode --input config.json --type common.Config --output config.pb
configtxlator proto_encode --input modified_config.json --type common.Config --output modified_config.pb
configtxlator compute_update --channel_id channel1 --original config.pb --updated modified_config.pb --output config_update.pb
```

channel_update.pb라는 이름의 새 protobuff에는 채널 구성에 적용해야 하는 앵커 피어 업데이트가 포함되어 있다. 우리는 채널 구성 업데이트 트랜잭션을 만들기위해 구성 업데이트를 트랜잭션에 포장할 수 있다.

```
configtxlator proto_decode --input config_update.pb --type common.ConfigUpdate --output config_update.json
echo '{"payload":{"header":{"channel_header":{"channel_id":"channel1", "type":2}},"data":{"config_update":'$(cat config_update.json)'}}}' | jq . > config_update_in_envelope.json
configtxlator proto_encode --input config_update_in_envelope.json --type common.Envelope --output config_update_in_envelope.pb
```

이제 config_update_in_envelope.pb채널을 업데이트하는 데 사용할 수있는 최종 아티팩트 를 사용할 수 있다. test-network디렉토리로 다시 이동한다.

```
cd ..
```

명령에 새 채널 구성을 제공하여 앵커 피어를 추가 할 수 있다. Org1에만 영향을 미치는 채널 구성 섹션을 업데이트하고 있으므로 다른 채널 구성원은 채널 업데이트를 승인할 필요가 없다.

```
peer channel update -f channel-artifacts/config_update_in_envelope.pb -c channel1 -o localhost:7050  --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

![image](https://user-images.githubusercontent.com/68358404/122313972-3ba6ff00-cf52-11eb-8468-a936dc15926d.png)

Org2에 대한 앵커 피어를 설정할 수 있다.

```
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
```

이제 채널의 두 번째 블록인 최신 채널 구성 블록을 가져온다.

```
peer channel fetch config channel-artifacts/config_block.pb -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com -c channel1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

channel-artifacts디렉토리로 다시 이동

```
cd channel-artifacts
```

다음 구성 블록을 디코딩하고 복사

```
configtxlator proto_decode --input config_block.pb --type common.Block --output config_block.json
jq '.data.data[0].payload.data.config' config_block.json > config.json
cp config.json config_copy.json
```

채널 구성에서 앵커 피어로 채널에 결합 된 Org2 피어를 추가

```
jq '.channel_group.groups.Application.groups.Org2MSP.values += {"AnchorPeers":{"mod_policy": "Admins","value":{"anchor_peers": [{"host": "peer0.org2.example.com","port": 9051}]},"version": "0"}}' config_copy.json > modified_config.json
```

이제 원래 및 업데이트 된 채널 구성을 다시 protobuf 형식으로 변환하고 그 차이를 계산할 수 있다.

```
configtxlator proto_encode --input config.json --type common.Config --output config.pb
configtxlator proto_encode --input modified_config.json --type common.Config --output modified_config.pb
configtxlator compute_update --channel_id channel1 --original config.pb --updated modified_config.pb --output config_update.pb
```

구성 업데이트를 트랜잭션에 래핑하여 채널 구성 업데이트 트랜잭션을 만든다.

```
configtxlator proto_decode --input config_update.pb --type common.ConfigUpdate --output config_update.json
echo '{"payload":{"header":{"channel_header":{"channel_id":"channel1", "type":2}},"data":{"config_update":'$(cat config_update.json)'}}}' | jq . > config_update_in_envelope.json
configtxlator proto_encode --input config_update_in_envelope.json --type common.Envelope --output config_update_in_envelope.pb
```

test-network디렉토리로 다시 이동

```
cd ..
```

다음 명령을 실행하여 채널을 업데이트하고 Org2 앵커 피어를 설정.

```
peer channel update -f channel-artifacts/config_update_in_envelope.pb -c channel1 -o localhost:7050  --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

![image](https://user-images.githubusercontent.com/68358404/122314318-f0412080-cf52-11eb-9e9e-ae4e028188a6.png)


채널이 성공적으로 업데이트되었는지 확인

```
peer channel getinfo -c channel1
```

채널 생성 블록에 두 개의 채널 구성 블록을 추가하여 채널이 업데이트되었으므로 채널의 높이가 3 개로 늘어났다.

![image](https://user-images.githubusercontent.com/68358404/122314357-00590000-cf53-11eb-96b4-c627ba6b391e.png)

## 9-8. 새 채널에 체인 코드 배포

채널에 체인 코드를 배포하여 채널이 성공적으로 생성되었는지 확인할 수 있다.

```
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go/ -ccl go -c channel1 -cci InitLedger
```

![image](https://user-images.githubusercontent.com/68358404/122314520-49a94f80-cf53-11eb-8c17-9ecdef57e986.png)


질의를 통해 데이터가 추가되었음을 확인할 수 있다.

```
peer chaincode query -C channel1 -n basic -c '{"Args":["getAllAssets"]}'
```

쿼리를 실행 한 후 채널 원장에 추가 된 자산이 표시

![image](https://user-images.githubusercontent.com/68358404/122314538-57f76b80-cf53-11eb-9afa-7a58191f4e51.png)

## 9-9. configtx.yaml을 사용하여 채널 구성 빌드

 채널은 채널의 초기 구성을 지정하는 채널 생성 트랜잭션 아티팩트를 구축하여 생성된다. 채널 생성 트랜잭션 파일을 수동으로 구축할 수 있지만 configtx.yaml파일 및 configtxgen 도구를 사용하여 채널을 생성하는 것이 더 쉽다. configtx.yaml파일을 쉽게 읽고 인간에 의해 편집 할 수있는 형식으로 채널 구성을 구축하는 데 필요한 정보가 포함되어 있습니다. 이 configtxgen도구는 configtx.yaml파일 의 정보를 읽고 Fabric에서 읽을 수있는 protobuf 형식으로 작성한다.

```
cd fabric-samples/test-network
```

configtx.yaml테스트 네트워크에서 사용하는 파일은에 위치한 configtx폴더에 있다. cd configtx 후 code . 로 파일을 열어보자. configtx.yaml은 조직,능력,신청,주문자,채널,프로필로 구성되어있다.


### 9-9-1. 조직(Organization)

 채널 구성에 포함 된 가장 중요한 정보는 채널 구성원인 조직이다. configtx.yaml테스트 네트워크에서 사용하는 파일은 세 개의 조직이 포함되어있다. 애플리케이션 채널에 추가 할 수 있는 피어조직 Org1과 Org2 그리고  OrdererOrg라는 Ordering Servcice 관리자다. 
 
 ![image](https://user-images.githubusercontent.com/68358404/122317120-f5549e80-cf57-11eb-8b0d-a4ef55834a70.png)

- Name 필드는 조직을 식별하는 데 사용되는 비공식적인 이름

- ID필드는 조직의 MSP ID입니다. MSP ID는 조직의 고유 식별자 역할을 하며 채널정책에 의해 참조되며 채널에 제출된 트랜잭션에 포함한다.

- MSPDir조직에 의해 생성 된 MSP 폴더의 경로이다. configtxgen도구는 채널 MSP를 만들려면이 MSP 폴더를 사용한다.

### 9-9-2. 능력(Capabilities)
 
 서로 다른 버전의 Hyperledger Fabric을 실행하는 orderer 및 피어 노드에 의해 결합 될 수 있다. 채널 기능을 사용하면 다른 패브릭 바이너리를 실행하는 조직이 특정기능만 활성화하여 동일한 채널에 참여할 수 있다. 예를 들어 Fabric v1.4를 실행하는 조직과 Fabric v2.x를 실행하는 조직은 채널 기능 수준이 V1_4_X 이하로 설정되어있는 한 동일한 채널에 참여할 수 있다.  Fabric v2.0에 도입 된 기능을 사용할 수 없다.
 
configtx.yaml파일 을 조사하면 세 가지 기능 그룹이 표시된다. 
 - 애플리케이션 기능은 Fabric 체인 코드 라이프 사이클과 같은 피어 노드가 사용하는 기능을 제어하고 채널에 조인 된 피어가 실행할 수있는 Fabric 바이너리의 최소 버전을 설정한다.
 - Orderer 기능은 Raft 합의와 같은 orderer 노드에서 사용되는 기능을 제어하고 채널 동의자 집합에 속하는 노드를 주문하여 실행할 수있는 Fabric 바이너리의 최소 버전을 설정한다.
 - 채널 기능은 피어 및 주문 노드에서 실행할 수있는 패브릭의 최소 버전을 설정한다.
 
### 9-9-3. 어플리케이션
 
 애플리케이션 섹션은 피어 조직이 애플리케이션 채널과 상호 작용할 수있는 방법을 제어하는 정책을 정의합니다. 이러한 정책은 체인 코드 정의를 승인하거나 채널 구성 업데이트 요청에 서명해야하는 피어 조직의 수를 제어합니다. 이러한 정책은 채널 원장에 기록하거나 채널 이벤트를 쿼리하는 기능과 같은 채널 리소스에 대한 액세스를 제한하는데도 사용된다.
  
 테스트 네트워크는 Hyperledger Fabric에서 제공하는 기본 애플리케이션 정책을 사용한다. 기본 정책을 사용하면 모든 피어 조직이 원장에서 데이터를 읽고 쓸 수 있다. 또한 기본 정책에서는 채널 구성원의 대다수가 채널 구성 업데이트에 서명하고 체인 코드를 채널에 배포하기 전에 대부분의 채널 구성원이 체인 코드 정의를 승인해야한다.
  
## 9-9-4. Orderer
  
 각 채널 구성에는 채널 동의자 집합의 주문자 노드가 포함된다. 동의자 집합은 새 블록을 생성하고 채널에 가입 한 피어에게 배포 할 수있는 order 노드 그룹입니다. 동의자 집합의 구성원인 각 순서 지정 노드의 끝점(Endpoint) 정보는 채널 구성에 저장된다.
  
 Orderer 섹션 configtx.yaml을 사용하여 단일 노드 Raft 주문 서비스를 만든다.
 OrdererType필드는 합의 유형으로 Raft를 선택하는 데 사용
 Raft Order 서비스는 합의 프로세스에 참여할 수있는 동의자 목록에 의해 정의됩니다. 테스트 네트워크는 단일 정렬 노드 만 사용하므로 동의자 목록에는 엔드 포인트가 하나만 포함됩니다.
 
 ![image](https://user-images.githubusercontent.com/68358404/122318822-ab20ec80-cf5a-11eb-8bb5-8b900f9b8902.png)
 
- 동의자 목록의 각 순서 지정 노드는 엔드 포인트 주소와 클라이언트 및 서버 TLS 인증서로 식별된다. 다중 노드 순서 지정 서비스를 배포하는 경우 호스트 이름, 포트 및 각 노드에서 사용하는 TLS 인증서에 대한 경로를 제공해야한다. 또한 각 순서 지정 노드의 엔드 포인트 주소를의 목록(Addresses)에 추가해야한다.

- BatchTimeout및 BatchSize필드를 사용하여 각 블록의 최대 크기와 새 블록이 생성되는 빈도를 변경하여 채널의 지연 시간과 처리량을 조정할 수 있습다.

- Policies 섹션에서는 채널 동의자 집합을 제어하는 정책을 만듭니다. 테스트 네트워크는 Fabric에서 제공하는 기본 정책을 사용하므로 대부분의 주문 관리자가 주문 노드, 조직의 추가 또는 제거를 승인하거나 블록 절단 매개 변수에 대한 업데이트를 승인해야한다.

## 9-9-4. 채널

 채널 섹션은 채널 구성의 최고 수준을 관리하는 정책을 정의한다. 애플리케이션 채널의 경우 이러한 정책은 해싱 알고리즘, 새 블록을 만드는 데 사용되는 데이터 해싱 구조 및 채널 기능 수준을 제어한다. 시스템 채널에서 이러한 정책은 또한 피어 조직의 컨소시엄 생성 또는 제거를 제어한다.
 
 테스트 네트워크는 Fabric에서 제공하는 기본 정책을 사용하므로 대부분의 주문자 서비스 관리자가 시스템 채널에서 이러한 값에 대한 업데이트를 승인해야한다. 애플리케이션 채널에서 변경 사항은 대다수의 Orderer 조직과 다수의 채널 구성원의 승인을 받아야한다. 대부분의 사용자는이 값을 변경할 필요가 없다.
 

## 9-9-5. 프로필

 configtxgen도구는 프로필 섹션 에서 채널 프로필을 읽어 채널 구성을 만든다. 각 프로필은 YAML 구문을 사용하여 파일의 다른 섹션에서 데이터를 수집한다. 이 configtxgen도구는이 구성을 사용하여 애플리케이션 채널에 대한 채널 생성 트랜잭션을 생성하거나 시스템 채널에 대한 채널 생성 블록을 작성한다.
 
configtx.yaml테스트 네트워크에 의해 사용되는 두 채널(TwoOrgsOrdererGenesis, TwoOrgsChannel) 정보를 포함한다.

- TwoOrgsOrdererGenesis

![image](https://user-images.githubusercontent.com/68358404/122321046-33ed5780-cf5e-11eb-95b1-662f5b80734e.png)

시스템 채널은 Order 서비스의 노드와 서비스 관리자를 주문하는 조직 집합을 정의한다.
시스템 채널에는 블록 체인 컨소시엄에 속한 피어 조직 세트도 포함된다. 

Organizations : 섹션 의 OrdererOrg 는 주문 서비스의 유일한 관리자가 된다.

- TwoOrgsChannel

  TwoOrgsChannel프로필은 테스트 네트워크에서 애플리케이션 채널을 만드는 데 사용된다.
  시스템 채널은 Order 서비스에서 애플리케이션 채널을 생성하기위한 템플릿으로 사용됩니다. 시스템 채널에 정의 된 주문 서비스의 노드는 새 채널의 기본 동의자 세트가되고, 주문 서비스의 관리자는 채널의 주문자 관리자가 된다.
  
  
## 9-10. 채널 정책

 채널은 조직 간의 개인 통신 방법이다. 따라서 채널 구성에 대한 대부분의 변경 사항은 채널의 다른 구성원이 동의 해야한다. 조직이 다른 조직의 승인없이 채널에 참여하여 원장의 데이터를 읽을 수 있다면 채널은 유용하지 않다. 채널 구조에 대한 모든 변경 사항 은 채널 정책을 충족 할 수있는 일련의 조직에서 승인해야다.

 정책은 또한 채널에 배포하기 전에 체인 코드를 승인해야하는 조직 집합이나 채널 관리자가 완료 해야하는 작업과 같이 사용자가 채널과 상호작용하는 방식의 프로세스를 제어한다.
 
 채널 구성의 다른 부분과 달리 채널을 제어하는 정책은 configtx.yaml 파일 의 여러 섹션이 함께 작동 하는 방식에 따라 결정된다.  제약조건이 거의없는 모든 사용 사례에 대해 채널 정책을 구성 할 수 있지만, 이 항목에서는 Hyperledger Fabric에서 제공하는 기본 정책을 사용하는 방법에 중점을 둔다.

## 9-11. 서명 정책

 기본적으로 각 채널 구성원은 조직을 참조하는 서명 정책 세트를 정의한다. 제안이 피어에 제출되거나 트랜잭션이 주문 노드에 제출되면 노드는 트랜잭션에 첨부 된 서명을 읽고 채널 구성에 정의된 서명 정책에 대해 평가한다. 모든 서명 정책에는 서명이 정책을 충족할 수있는 조직 및 ID 집합을 지정하는 규칙이 있다. 아래의 조직 섹션 에서 Org1에 의해 정의 된 서명 정책을 볼 수 있다.
 
![image](https://user-images.githubusercontent.com/68358404/122322716-e0303d80-cf60-11eb-90f1-96588f311758.png)

 위의 모든 정책은 Org1의 서명을 통해 충족할 수 있습니다. 그러나 각 정책에는 정책을 충족할 수 있는 조직 내에서 다른 역할 집합이 나열됩니다. 관리자 정책은 관리자 역할을 가진 ID가 제출한 트랜잭션을 통해서만 충족할 수 있으며 피어 역할을 가진 ID만 승인 정책을 충족할 수 있습니다. 단일 트랜잭션에 연결된 서명 집합은 여러 서명 정책을 충족할 수 있습니다. 예를 들어, 트랜잭션에 첨부된 보증이 Org1과 Org2에 의해 제공되었다면, 이 서명 세트는 Org1과 Org2의 보증 정책을 충족할 것이다.
 
## 9-12. ImplicitMeta 정책
 
 채널에서 기본 정책을 사용하는 경우 각 조직의 서명 정책은 채널 구성의 상위 수준에서 ImplicitMeta 정책에 의해 평가된다. 채널에 제출 된 서명을 직접 평가하는 대신 ImplicitMeta 정책에는 정책을 충족 할 수있는 채널 구성의 다른 정책 집합을 지정하는 규칙이 있다. 트랜잭션은 정책에서 참조하는 기본 서명 정책 집합을 충족 할 수있는 경우 ImplicitMeta 정책을 충족 할 수 있습니다.
 
![image](https://user-images.githubusercontent.com/68358404/122324987-bc6ef680-cf64-11eb-8285-86cf7683fb81.png)

애플리케이션 섹션 의 ImplicitMeta 정책은 피어 조직이 채널과 상호 작용하는 방식을 제어한다.
애플리케이션 섹션의 정책과 아래 조직 섹션 의 정책 간의 관계를 볼 수 있다.

![image](https://user-images.githubusercontent.com/68358404/122325051-d7da0180-cf64-11eb-8596-ca0f17cd372e.png)

## 9-13. 채널 수정 정책

 채널구조는 채널 구성 내의 수정정책에 의해 관리됩니다. 채널 구성의 각 구성요소에는 채널 구성원이 업데이트하기 위해 충족해야하는 수정 정책이 있다. 예를 들어, 각 조직에서 정의한 정책 및 채널 MSP, 채널 구성원을 포함하는 애플리케이션 그룹 및 채널 동의자 세트를 정의하는 구성의 구성 요소는 각각 다른 수정 정책을 갖습니다.
 
## 9-14. 채널 정책 및 액세스 제어 목록

 채널 구성 내의 정책은 채널에서 사용하는 패브릭 리소스에 대한 액세스를 제한하는 데 사용되는 ACL (액세스 제어 목록) 에서도 참조된다. ACL은 채널 구성 내의 정책을 확장하여 채널의 프로세스 를 관리한다.
 
## 9-16. 주문자 정책
 Orderer 섹션 에있는 ImplicitMeta 정책 configtx.yaml은 Application 섹션이 피어 조직을 관리 하는 것과 유사한 방식으로 채널의 순서 지정 노드를 관리한다. ImplicitMeta 정책은 서비스 관리자를 주문하는 조직과 관련된 서명 정책을 가리킨다.
 
![image](https://user-images.githubusercontent.com/68358404/122332997-6608b480-cf72-11eb-974c-766cb4d24e88.png)

기본 정책을 사용하는 경우 대부분의 주문자 조직은 주문 노드의 추가 또는 제거를 승인해야한다.

![image](https://user-images.githubusercontent.com/68358404/122333021-73be3a00-cf72-11eb-9e96-1a12a38ea155.png)

채널에서 주문 노드를 제거하기 위해 제출 된 요청에는 채널 / 주문자 / 관리자 정책을 충족하는 네트워크의 세 가지 주문 조직의 서명이 포함되어 있다. 서명이 과반수에 도달 할 필요가 없었기 때문에 Org3 검사는 연한 녹색으로 표시된다.

Channel/Orderer/BlockValidation정책은 피어가 채널에 추가되는 새 블록이 채널 동의자 집합의 일부인 주문 노드에 의해 생성되었으며 블록이 다른 피어 조직에 의해 변조되거나 생성되지 않았음을 확인하는 데 사용된다. 기본적으로 Writers서명 정책 이있는 모든 주문자 조직 은 채널에 대한 블록을 만들고 유효성을 검사 할 수 있다.

## 10-1. 채널에 조직 추가

  이 튜토리얼은 애플리케이션 채널에 새로운 조직 (Org3)을 추가하여 패브릭 테스트 네트워크를 확장한다. 
  
## 10-2. 환경 설정

테스트 네트워크 channel1 생성

```
cd fabric-samples/test-network
./network.sh down
./network.sh up createChannel -c channel1
```

## 10-3. 스크립트를 사용하여 Org3을 채널로 가져오기

```
cd addOrg3
./addOrg3.sh up -c channel1
```

Org3 암호화 자료가 생성되고 Org3 조직 정의가 생성된 다음 채널 구성이 업데이트되고 서명된 다음 채널에 제춸되는 것을 볼 수 있다.

이제 채널에 Org3를 추가할 수 있음을 확인했으므로 스크립트가 백그라운드에서 완료한 채널 구성을 업데이트하는 단계를 수행 할 수 있다.

## 10-4. Org3을 채널에 수동으로 가져오기

 방금 addOrg3.sh스크립트를 사용한 경우 네트워크를 중단해야합니다. 다음 명령은 실행중인 모든 구성 요소를 종료하고 모든 조직의 암호화 자료를 제거한다.

```
cd ..
./network.sh down
```

네트워크가 중단되면 다시 활성화한다.

```
./network.sh up createChannel -c channel1
```

그러면 네트워크가 addOrg3.sh스크립트 를 실행하기 전과 동일한 상태로 돌아간다.
이제 수동으로 Org3를 채널에 추가 할 준비가되었다. 첫 번째 단계로 Org3의 암호화 자료를 생성한다.

## 10-5. Org3 암호화 자료 생성

```
cd addOrg3
```
 
먼저 응용 프로그램 및 관리자 사용자와 함께 Org3 피어에 대한 인증서 및 키를 생성한다. 예제 채널을 업데이트하고 있기 때문에 인증 기관을 사용하는 대신 cryptogen 도구를 사용할 것이다. 
 다음 명령은 org3-crypto.yaml cryptogen을 사용하여 파일을 읽고 새 org3.example.com폴더 에 Org3 암호화 자료를 생성한다.
 
```
../../bin/cryptogen generate --config=org3-crypto.yaml --output="../organizations"
```

test-network/organizations/peerOrganizations 디렉토리에서 Org1 및 Org2에 대한 인증서 및 키와 함께 생성 된 Org3 암호화 자료를 찾을 수 있다.

 Org3 암호화 자료를 생성했으면 configtxgen 도구를 사용하여 Org3 조직 정의를 인쇄 할 수 있습니다. configtx.yaml가 수집해야하는 파일을 현재 디렉토리에서 찾도록 지시하여 명령을 시작한다.
 
```
export FABRIC_CFG_PATH=$PWD
../../bin/configtxgen -printOrg Org3MSP > ../organizations/peerOrganizations/org3.example.com/org3.json
```

![image](https://user-images.githubusercontent.com/68358404/122335050-a584d000-cf75-11eb-84d9-5458a9442774.png)

위의 명령은 JSON 파일을 만들고 – org3.json– test-network/organizations/peerOrganizations/org3.example.com 폴더에 쓴다. 조직 정의에는 Org3에 대한 정책 정의, Org3에 대한 NodeOU 정의 및 base64 형식으로 인코딩 된 두 개의 중요한 인증서가 포함된다.

* 조직의 신뢰 루트를 설정하는 데 사용되는 CA 루트 인증서
* 블록 보급 및 서비스 검색을 위해 Org3를 식별하기 위해 가십 프로토콜에서 사용하는 TLS 루트 인증서

이 조직 정의를 채널 구성에 추가하여 Org3을 채널에 추가한다.

## 10-6. Org3 구성 요소 불러오기

Org3 인증서 자료를 만든 후 이제 Org3 피어를 가져올 수 있습니다. 로부터 addOrg3디렉토리, 다음 명령을 실행

```
docker-compose -f docker/docker-compose-org3.yaml up -d
```

명령이 성공하면 Org3 피어가 생성 된 것을 볼 수 있다.

![image](https://user-images.githubusercontent.com/68358404/122336031-27c1c400-cf77-11eb-9c94-fbeadc1248de.png)

----------------------------------------------

-tip 밑의 사진과 에러가 발생 시

![image](https://user-images.githubusercontent.com/68358404/122336072-37d9a380-cf77-11eb-9951-d4d4258664cf.png)

```
export IMAGE_TAG=latest
docker-compose -f docker/docker-compose-org3.yaml up -d
```

--------------------------------------------------


## 10-7. 구성 가져오기

 채널에 대한 가장 최근의 구성 블록을 가져온다. 최신 버전의 구성을 가져와야하는 이유는 채널 구성 요소의 버전이 지정 되었기 때문입니다. 버전 관리는 여러 가지 이유로 중요합니다. 구성 변경이 반복되거나 재생되는 것을 방지한다. 또한 동시성을 보장하는 데 도움이 된다.
 
 Org3은 아직 채널의 구성원이 아니기 때문에 채널 구성을 가져 오려면 다른 조직의 관리자로 작동해야한다. Org1은 채널의 구성원이므로 Org1 관리자는 주문 서비스에서 채널 구성을 가져올 수있는 권한이 있다. Org1 관리자로 작동하려면 다음 명령을 실행한다.
 
```
cd ../
export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=${PWD}/../config/
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```

이제 최신 구성 블록을 가져 오는 명령을 실행가능

```
peer channel fetch config channel-artifacts/config_block.pb -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com -c channel1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

이 명령은 바이너리 protobuf 채널 구성 블록을 config_block.pb에 저장한다. 이름과 파일 확장자는 임의로 선택할 수 있습니다. 그러나 표현되는 객체 유형과 인코딩 (protobuf 또는 JSON)을 모두 식별하는 규칙을 따르는 것이 좋다.

![image](https://user-images.githubusercontent.com/68358404/122336426-d108ba00-cf77-11eb-947b-26ff630d7fe4.png)

 채널 1에 대한 가장 최근의 구성 블록이 실제로 genesis 블록이 아닌 블록 2라는 것을 말해준다. 기본적으로 peer channel fetch config 명령은 대상 채널에 대한 최신 구성 블록을 반환한다. 이 경우 세 번째 블록이다. 이는 테스트 네트워크 스크립트인 network.sh이 두 개의 별도 채널 업데이트 트랜잭션에서 Org1과 Org2의 앵커 피어를 정의했기 때문이다. 그 결과 다음과 같은 구성 시퀀스가 제공된다.

블록 0: 제네시스 블록
블록 1: Org1 앵커 피어 업데이트
블록 2: Org2 앵커 피어 업데이트

## 10-7. 구성을 JSON으로 변환하고 잘라내기

 채널 구성 블록은 channel-artifacts 다른 아티팩트와 별도로 업데이트 프로세스를 유지하기 위해 폴더에 저장되었다 . channel-artifacts 다음 단계를 완료 하려면 폴더로 변경한다.
 configtxlator도구를 사용하여 채널 구성 블록을 JSON 형식 (사람이 읽고 수정할 수 있음)으로 디코딩한다. 또한 변경하려는 변경과 관련이없는 헤더, 메타 데이터, 작성자 서명 등을 모두 제거해야한다.
 
```
cd ./channel-artifacts
configtxlator proto_decode --input config_block.pb --type common.Block --output config_block.json
jq .data.data[0].payload.data.config config_block.json > config.json
```

이 명령 config.json을 사용하면 구성 업데이트의 기준이 될 JSON 개체가 줄어든다. vscode로 한번 확인

## 10-8. Org3 암호화 자료 추가
 
jq도구를 다시 한 번 사용하여 Org3 구성 정의 Org3.json을 채널의 애플리케이션 그룹 필드에 추가하고 출력 이름을 –modified_config.json.으로 지정한다.

```
jq -s '.[0] * {"channel_group":{"groups":{"Application":{"groups": {"Org3MSP":.[1]}}}}}' config.json ../organizations/peerOrganizations/org3.example.com/org3.json > modified_config.json
```

이제 관심 있는 두 개의 JSON 파일(config.json 및 modified_config.json)이 있다. 초기 파일은 Org1과 Org2 재료만 포함하지만, "수정된" 파일은 3개의 조직이 모두 포함된다. 이 시점에서는 이 두 JSON 파일을 다시 인코딩하고 델타를 계산하기만 하면 된다.

먼저 config.json을 config.pb라는 protobuf로 다시 변환한다.

```
configtxlator proto_encode --input config.json --type common.Config --output config.pb
```

다음으로 modified_config.json을 modified_config.pb로 인코딩한다.

```
configtxlator proto_encode --input config.json --type common.Config --output config.pb

```


다음, 인코딩 modified_config.json에 modified_config.pb :

```
configtxlator proto_encode --input modified_config.json --type common.Config --output modified_config.pb
```

이제 configtxlator를 사용하여이 두 구성 protobufs 간의 델타를 계산합니다. 이 명령은 org3_update.pb라는 새 protobuf 바이너리를 출력한다.

```
configtxlator compute_update --channel_id channel1 --original config.pb --updated modified_config.pb --output org3_update.pb
```

 이 새로운 proto – org3_update.pb –에는 Org3 정의와 Org1 및 Org2 자료에 대한 상위 수준 포인터가 포함되어 있다. Org1 및 Org2에 대한 광범위한 MSP 자료 및 수정 정책 정보는이 데이터가 이미 채널의 제네시스 블록 내에 존재하기 때문에 무시할 수 있습니다. 따라서 두 구성 사이의 델타 만 필요합니다.

채널 업데이트를 제출하기 전에 몇 가지 마지막 단계를 수행해야한다. 먼저 이 객체를 편집 가능한 JSON 형식으로 디코딩하고 이름을 org3_update.json으로 지정한다.

```
configtxlator proto_decode --input org3_update.pb --type common.ConfigUpdate --output org3_update.json
```

이제, 우리는 봉투(envelope) 메시지로 포장해야 하는 디코딩된 업데이트 파일(org3_update.json)을 가지고 있다. 이 단계를 통해 이전에 제거했던 헤더 필드를 다시 얻을 수 있습니다. 이 파일의 이름을 org3_update_in_envelope.json으로 지정한다.

```
echo '{"payload":{"header":{"channel_header":{"channel_id":"'channel1'", "type":2}},"data":{"config_update":'$(cat org3_update.json)'}}}' | jq . > org3_update_in_envelope.json
```

올바르게 구성된 JSON(org3_update_in_envelope.json)을 사용하여 configtxlator 도구를 마지막으로 한 번 더 활용하고 Fabric에 필요한 전체 프로토버프 형식으로 변환합니다. 최종 업데이트 개체의 이름을 org3_update_in_envelope.pb:

```
configtxlator proto_encode --input org3_update_in_envelope.json --type common.Envelope --output org3_update_in_envelope.pb
```

## 10-9. 구성 업데이트 서명 및 제출

 거의 완료되었다. protobuf 바이너리 – org3_update_in_envelope.pb가 있다. 그러나 구성을 원장에 기록하려면 필수 관리자 사용자의 서명이 필요하다. 채널 애플리케이션 그룹에 대한 수정 정책 (mod_policy)은 기본값 인 "MAJORITY"로 설정 되어있으며 이는 기존 조직 관리자의 대다수가 서명해야 함을 의미한다. Org1과 Org2라는 두 개의 조직만 있고 두 개의 대부분이 2 개이므로 서명하려면 둘 다 필요합니다. 두 서명이 모두 없으면 order 서비스는 정책을 이행하지 못한 트랜재션을 거부합니다.

먼저이 업데이트 proto에 Org1으로 서명하겠다. 테스트 네트워크 디렉토리로 다시 이동한다.

Org1 관리자로 작동하는데 필요한 환경 변수를 설정한다. 결과적으로 다음 peer channel signconfigtx 명령은 업데이트에 Org1로 서명한다.

```
cd ../
peer channel signconfigtx -f channel-artifacts/org3_update_in_envelope.pb
```

![image](https://user-images.githubusercontent.com/68358404/122346312-c7854f00-cf83-11eb-9a9d-6f0a677ebeb6.png)

마지막 단계는 Org2 Admin 사용자를 반영하도록 컨테이너의 ID를 전환하는 것입니다. Org2 MSP에 특정한 네 가지 환경 변수를 내 보내면 된다.

```
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
```

마지막으로 명령을 내립니다. Org2 Admin 서명이이 호출에 첨부되므로 protobuf에 두 번째로 수동으로 서명할 필요가 없다.

```
peer channel update -f channel-artifacts/org3_update_in_envelope.pb -c channel1 -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

![image](https://user-images.githubusercontent.com/68358404/122346591-192dd980-cf84-11eb-912a-b05e7cefe756.png)

 성공적인 채널 업데이트 호출은 채널의 모든 피어에 새 블록 (블록 3)을 반환한다. 기억한다면 블록 0-2는 초기 채널 구성이다. 블록 3은 현재 채널에 정의된 Org3과 함께 최신 채널 구성으로 사용된다.

다음 명령을 실행하여 peer0.org1.example.com의 로그를 검사 할 수 있습니다.

```
docker logs -f peer0.org1.example.com
```

## 10-10. 채널에 Org3 가입

이 시점에서 채널 구성은 새로운 조직인 Org3을 포함하도록 업데이트되었습니다. 즉, 연결된 피어가 이제 channel1에 가입 할 수 있습니다. Org3 관리자로 작동하도록 다음 환경 변수 설정.

```
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org3MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org3.example.com/users/Admin@org3.example.com/msp
export CORE_PEER_ADDRESS=localhost:11051
```
