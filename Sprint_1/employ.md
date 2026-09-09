## git clone https://github.com/OT-MICROSERVICES/employee-api.git
cd employee-ap

## 3. scylladb

curl -sSf get.scylladb.com/server | sudo bash
sudo scylla_dev_mode_setup --developer-mode 1
sudo systemctl enable scylla-server
sudo systemctl start scylla-serve

cqlsh 127.0.0.1 9042

CREATE KEYSPACE employee_db WITH replication = {
    'class': 'NetworkTopologyStrategy', 
    'replication_factor': 1
};


## 2. Redis Cache Setup
sudo apt install redis-server -y
sudo systemctl start redis-server
sudo systemctl enable redis-server

## go

wget https://dl.google.com/go/go1.20.14.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.20.14.linux-amd64.tar.gz

 # 1. Extract karke /usr/local me install karein
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.20.14.linux-amd64.tar.gz

# 2. PATH export karein taaki terminal Go recognize kar sake
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc

# 3. Version verify karein
go version

### vim config.yaml vim config.yaml
ip change docker to localhost

### vim migration.json
ip change docker to localhost

### vim main.go 
45 line me yy add kena h iska = url := ginSwagger.URL("/swagger/doc.json")

 ### vim docs/docs.go 
 Host:             "(ip-server):8080",

### migration
curl -L https://github.com/golang-migrate/migrate/releases/download/v4.15.2/migrate.linux-amd64.tar.gz | tar xvz
sudo mv migrate /usr/local/bin/
migrate -version

 sudo apt install make
make run-migrations

## 
# 1. Ensure karein ki main.go ka syntax fix ho gaya hai, phir build karein
make build

# 2. Background me run karein
nohup ./employee-api > ~/employee.log 2>&1 &

# 3. Check karein ki port 8080 par listener active ho gaya ya nahi
ss -tulnp | grep 8080

go mod tidy
 make build
  nohup ./employee-api > ~/employee.log 2>&1 &
  
curl http://localhost:8080/api/v1/employee/health/detail

http://43.204.108.146:8080/swagger/doc.json


http://43.204.108.146:8080/swagger/index.html
