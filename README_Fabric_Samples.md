
# Hướng Dẫn Triển Khai Hyperledger Fabric Sample

Tài liệu này hướng dẫn cài đặt môi trường và triển khai **Hyperledger Fabric Sample Test Network** trên hệ điều hành Ubuntu (>= Ubuntu 22.04.5 LTS).

---

## 1. Cài Đặt Phần Mềm Cần Thiết

```bash
sudo apt-get install git
sudo apt-get install curl
sudo apt-get -y install docker-compose
```

Kiểm tra phiên bản:

```bash
docker --version
docker-compose --version
```

Bật Docker khi khởi động:

```bash
sudo systemctl enable docker
```

Thêm user hiện tại vào group Docker (thay `phongnh` bằng tên user của bạn):

```bash
sudo usermod -a -G docker phongnh
```

---

## 2. Cài Đặt Golang

```bash
wget https://dl.google.com/go/go1.24.3.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.24.3.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
source ~/.bashrc
go version
```

---

## 3. Tải Fabric Script và Sample Repo

```bash
mkdir -p $HOME/go/src/github.com/<your_github_userid>
cd $HOME/go/src/github.com/<your_github_userid>
curl -sSLO https://raw.githubusercontent.com/hyperledger/fabric/main/scripts/install-fabric.sh
chmod +x install-fabric.sh
./install-fabric.sh -h
```

Ví dụ chạy script để pull Docker containers và clone sample repo:

```bash
./install-fabric.sh docker samples binary
# hoặc để chỉ định version:
./install-fabric.sh --fabric-version 2.5.13 binary
```

---

## 4. Chạy Test Network

```bash
cd fabric-samples/test-network

# Dọn dẹp nếu đã từng chạy
./network.sh down

# Khởi động network
./network.sh up

# Kiểm tra container
docker ps -a

# Tạo channel và deploy chaincode
./network.sh createChannel
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
```

---

## 5. Thiết Lập Biến Môi Trường và Giao Tiếp Mạng

### Biến Môi Trường Chung

```bash
export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=$PWD/../config/
```

### Org1 Environment

```bash
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```

### Invoke InitLedger

```bash
peer chaincode invoke -o localhost:7050 \
--ordererTLSHostnameOverride orderer.example.com \
--tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" \
-C mychannel -n basic \
--peerAddresses localhost:7051 \
--tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" \
--peerAddresses localhost:9051 \
--tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" \
-c '{"function":"InitLedger","Args":[]}'
```

### Query GetAllAssets

```bash
peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
```

### Invoke TransferAsset

```bash
peer chaincode invoke -o localhost:7050 \
--ordererTLSHostnameOverride orderer.example.com \
--tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" \
-C mychannel -n basic \
--peerAddresses localhost:7051 \
--tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" \
--peerAddresses localhost:9051 \
--tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" \
-c '{"function":"TransferAsset","Args":["asset6","Christopher"]}'
```

---

## 6. Giao Tiếp Với Org2

### Org2 Environment

```bash
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
```

### Query ReadAsset

```bash
peer chaincode query -C mychannel -n basic -c '{"Args":["ReadAsset","asset6"]}'
```

---

## 7. Tắt Mạng

```bash
./network.sh down
```

---

## Ghi Chú

- Đảm bảo `docker` hoạt động và user đã được thêm vào group Docker.
- Có thể cần đăng xuất và đăng nhập lại sau khi thêm user vào group.
- Nếu gặp lỗi, kiểm tra kỹ các biến môi trường và logs từ Docker container.
