# Simulating Mobility Patterns in OAI through Channel Modifications via Telnet

# Contact
For help and information for this project refer to this email : alexstolt@gmail.com

# Overview
This project focuses on running experiments in OpenAirInterface to simulate user movement by varying path loss parameters in the integrated channel models.


# Getting Started
## Clone the official repository 


```git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git```

## Checkout on a recent tag that we verified everything works as expected 
```
cd openairinterface5g 
git checkout ba2d7aad18788c2572cb0b96488dc96ef4089c83
```

## Build the correct version of DPDK 
```
curl -O https://fast.dpdk.org/rel/dpdk-20.11.9.tar.xz
tar -xf dpdk-20.11.9.tar.xz
cd dpdk-stable-20.11.9/
sudo apt-get install python3 python3-pip python3-setuptools python3-wheel ninja-build libnuma-dev
pip3 install meson pyelftools
meson setup build
cd build
ninja
sudo pip3 install meson
meson install
```

## Install RAN and UE alongside RFsimulator and Telnet server
```
cd openairinterface5g/cmake_targets/
sudo ./build_oai -I --phy_simulators -w SIMU --nrUE --gNB --build-lib telnetsrv
```

## Install the CN

### Install Docker and Docker Compose
```
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo groupadd docker
sudo usermod -aG docker $USER
sudo reboot
```

### Pull the images from Dockerhub

```
docker login
docker pull mysql:8.0
docker pull oaisoftwarealliance/oai-amf:v2.0.0
docker pull oaisoftwarealliance/oai-smf:v2.0.0
docker pull oaisoftwarealliance/oai-upf:v2.0.0
docker pull oaisoftwarealliance/trf-gen-cn5g:focal
```


# Deploy 5G CN, RAN and a UE

## Deploy CN
```
cd openairinterface5g/ci-scripts/yaml_files/5g_rfsimulator
docker compose up -d mysql oai-amf oai-smf oai-upf oai-ext-dn
```

## Deploy without a channel model

### Deploy the gNB 
```
cd openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O <path-to-gnb.conf> -E --sa --gNBs.[0].min_rxtxtime 6 --rfsim
```

Note: Replace the ```<path-to-gnb.conf>``` with the path of gnb.conf (file we provide).


### Deploy the UE
```
cd openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-uesoftmodem --band 78 -E --sa --rfsim -r 106 --numerology 1 --ssb 516 -C 3319680000 --uicc0.imsi 208990100001101
```

### Verify end-to-end connectivity
```
ping -I oaitun_ue1 192.168.72.135
```


## Deploy with a channel model

### Deploy the gNB (Downling Channel Model)
```
cd openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O <path-to-gnb.conf> -E --sa --gNBs.[0].min_rxtxtime 6 --rfsim --rfsimulator.options chanmod --telnetsrv
```

Note: Replace the ```<path-to-gnb.conf>``` with the path of gnb.conf (file we provide).


### Deploy the UE (Uplink Channel Model)
```
cd openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-uesoftmodem --band 78 -E --sa --rfsim -r 106 --numerology 1 --ssb 516 -C 3319680000 --uicc0.imsi 208990100001101 --rfsimulator.options chanmod -O <path-to-nrue.uicc.conf> --telnetsrv
```

Note: Replace the ```<path-to-nrue.uicc.conf>``` with the path of nrue.uicc.conf (file we provide).

Note: The ```--telnetsrv``` should be executed either on the UE or the gNB and NOT on both.

### Verify end-to-end connectivity
```
ping -I oaitun_ue1 192.168.72.135
```

## Modyfying the Channel Model 
Note: Read [this](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/openair1/SIMULATION/TOOLS/DOC/channel_simulation.md) for more details
### Connect to the Telnet server 
```telnet 127.0.0.1 9090```

### Modify some of the parameters
```
channelmod modify 0 ploss 12
channelmod modify 0 noise_power_dB 3
```


# Project goal
1. Modify the pathloss paremeter for values ranging from 10 to 25 on the gNB (hint: telnet on the nr-softmodem) and describe what you observe  
1. Modify the pathloss paremeter for values ranging from 10 to 25 on the UE (hint: telnet on the nr-uesoftmodem) and describe what you observe  

