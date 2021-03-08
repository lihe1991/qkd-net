qkd-net, modifey by He Li
=======

Overview
--------

The OpenQKDNetwork project aims to provide a reference implementation of a [4-layer Quantum Key Distribution (QKD) network solution](https://openqkdnetwork.ca).  Two sample applications are also provided to show case how an application can fetch and make use of the keys generated by a QKD network.

A very flexible layered approach has been adopted adhering to [SOLID](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)) principles.

Microservice based architecture approach has been taken for software design and
and implementation. One benefit of it is to have flexibility in production deployment.

**Complied and tested successfully on Ubuntu 16.04 and Linux Mint 18.3 with Java 8.  Changes will be needed for working in other environments (OS, Java version, etc.)**

Layers
------

1. Key Management System (KMS)
2. Quantum Network Layer (QNL)
3. Quantum Link Layer (QLL)

The fourth layer is the User (or Host) Layer, which belongs to the client of this system, and thus is beyond the scope of this project.
QKD devices belong to the QLL, and may need a vendor specific glue layer of code in QLL to make it work with the other layers, before any international standard (e.g. ETSI, ITU-T) on the interface is widely adopted.

###  Key Management System (KMS)

The KMS layer provides an application interface for fetching keys for symmetric
cryptography. The details of how the key is used belong to the applications at the User layer.

KMS layer comprises the following microservices:

1. Configuration service
2. Registration Service
3. Authorization Service
4. KMS Service
5. KMS API Gateway Service
6. KMS QNL Service

### Quantum Network Layer (QNL)

QNL is the core compoent to extend the QKD technology from point-to-point to network level. It is the middle layer sandwiched between KMS layer and QLL (Quantum Link Layer).
QNL provides key blocks to KMS after fetching it from QLL. Thus, it keeps the KMS
layer QKD-technology-agnostic.  

QNL currently consists of a single microservice:

1. Key Routing Service

Key Routing Service contains a routing modual and key relaying modual, to react to the QKD network topology dynamics automatically and in a timely fashion.  It not only pushes key blocks to KMS QNL service for KMS
service, but also responds to pull requests for key blocks directly from KMS service.

### QKD Link Layer (QLL)

Ideally, QLL consists of working QKD devices. A QLL simulator is provided in this project for situations where real QKD devices based links are unavailable, e.g., a planned satellite based QKD link between two nodes that are geographically far away.

# How to Run The Project
## Prerequisites (Required)

Install following before proceeding.
Make sure the executables are in system path.

1. Java SDK 8 or later - http://www.oracle.com/technetwork/java/javase/downloads/index.html
2. Maven - https://maven.apache.org/ or apt-get, if on Ubuntu
3. screen - apt-get, if on Ubuntu
4. git - apt-get, if on Ubuntu

### Prerequisites for testing (Optional)

On a Linux(Ubuntu) system apt-get can be used to install the programs.

1. Curl
2. jq

### Quick start guide

On two Linux (Ubuntu) systems, please follow the steps below to get a two nodes system running quickly.

1. Assume IP address of **A** is **192.168.2.207** and IP address of **B** is **192.168.2.212**
2. On Ubuntu **A**
3. *git clone https://github.com/Open-QKD-Network/qkd-net.git*
4. *cd qkd-net*
5. *cd kms*
6. *./scripts/build*
7. *rm ~/.qkd* (Step 6 generates ~/.qkd directory)
8. *cd ..*
9. *tar xvf **qkd-kaiduan-a.tar***
10. *mv .qkd ~/*
11. On Ubuntu **A**, changes IP address of **B** in the **~/.qkd/qnl/routes.json**.
```json
{
  "adjacent": {
    "B": "192.168.2.212"
  },
  "nonAdjacent": {
  }
}
```
12. Run commands 3 to 8 on Ubuntu B
13. *tar xvf **qkd-kaiduan-b.tar***
14. *mv .qkd ~/*
15. On Ubuntu **B**, changes IP address of **A** in the **~/.qkd/qnl/routes.json**
```json
{
  "adjacent": {
    "A": "192.168.2.207"
  },
  "nonAdjacent": {
  }
}
```
16. On Ubuntu A, *cd qkd-net/kms*, run command *./scripts/run*
17. On Ubuntu B, *cd qkd-net/kms*, run command *./scripts/run*
18. We will run tls-demo application alice on Ubuntu A and run bob on Ubuntu B
19. On Ubuntu A, *cd ../applications/tls-demo/*, run *make* command
20. On Ubuntu B, *cd ../applications/tls-demo/*, change function static char* site_id(char* ip) in **src/bob.c** as below, and run *make*
```c
        static char* site_id(char* ip) {
            if (strcmp(ip, "192.168.2.212") == 0)
                return "B";
            else if (strcmp(ip, "192.168.2.207") == 0)
                return "A";
            else
                return "C";
        }
```
21. On Ubuntu B, run command *./bob -b 10446*
22. On Ubuntu A, run command *./alice -i 192.168.2.212 -b 10446 -f ./data/whale.mp3*
23. On Ubuntu A, you should see something like below
```
        -- Successfully conected to Bob
           -- Received PSK identity hint 'B c5460688-c6ff-4e0e-8c7c-7c82d18ce0cc 0'
        HTTP-FETCH-REQUEST,url:http://localhost:9992/uaa/oauth/token,header:authorization:Basic aHRtbDU6cGFzc3dvcmQ=,post:password=bot&client_secret=password&client=html5&username=pwebb&grant_type=password&scope=openid
        HTTP-FETCH-RESPONSE:{"access_token":"e5d40647-81c6-467b-98f2-54fe2e9086d5","token_type":"bearer","refresh_token":"fd26f0a0-8c30-43af-beae-700147bed","expires_in":43199,"scope":"openid"}
        post String siteid=B&index=0&blockid=c5460688-c6ff-4e0e-8c7c-7c82d18ce0cc
        HTTP-FETCH-REQUEST,url:http://localhost:8095/api/getkey,header:Authorization: Bearer e5d40647-81c6-467b-98f2-54fe2e9086d5,post:siteid=B&index=0&blockid=c5460688-c6ff-4e0e-8c7c-7c82d18ce0cc
        HTTP-FETCH-RESPONSE:{"index":0,"hexKey":"e27444fab22da64bb65dd108adc84f12a2a94a0fd5129e37df44025ca1622942","blockId":"c5460688-c6ff-4e0e-8c7c-7c82d18ce0cc"}
        index: 0
        key: e27444fab22da64bb65dd108adc84f12a2a94a0fd5129e37df44025ca1622942
        blockId: c5460688-c6ff-4e0e-8c7c-7c82d18ce0cc
```

24. On Ubuntu B, file is received and saved in **bobdemo** and should see something like below
```
        Bob is listening for incomming connections on port 10446 ...
        HTTP-FETCH-REQUEST, url:http://localhost:9992/uaa/oauth/token,header:authorization:Basic aHRtbDU6cGFzc3dvcmQ=,post:password=bot&client_secret=password&client=html5&ername=pwebb&grant_type=password&scope=openid
        HTTP-FETCH-RESPONSE:{"access_token":"24b4b875-1146-406d-84cf-4379a4da2b96","token_type":"bearer","refresh_token":"52f93b1b-b642-4884-96e3-0186aaf69381","expires_in":43199,"scope":"openid"}
        key_post : siteid=A
        HTTP-FETCH-REQUEST, url:http://localhost:8095/api/newkey,header:Authorization: Bearer 24b4b875-1146-406d-84cf-4379a4da2b96,post:siteid=A
        HTTP-FETCH-RESPONSE:{"index":0,"hexKey":"e27444fab22da64bb65dd108adc84f12a2a94a0fd5129e37df44025ca1622942","blockId":"c5460688-c6ff-4e0e-8c7c-7c82d18ce0cc"}
        index: 0
        key: e27444fab22da64bb65dd108adc84f12a2a94a0fd5129e37df44025ca1622942
        blockId: c5460688-c6ff-4e0e-8c7c-7c82d18ce0cc
        SSL-Server-PSK-Hint:B c5460688-c6ff-4e0e-8c7c-7c82d18ce0cc 0
        -- SHA1 of the received key : 8F23A277146D8DB4F9C4B6F320BE5D772396CD10
        -- TLS Closed
        -- Connection Closed
        -- Accept Connection
```
25. For 3 nodes system, extract **qkd-kaiduan-c.tar** on Ubuntu, assume we will run alice on A and bob on C, the network topology looks below,
> A  <----> B <---> C
26. Assume IP address of **C** is **192.168.2.235**, Add C to **B**'s adjacent nodes
```json
{
  "adjacent": {
    "A": "192.168.2.207",
    "C": "192.168.2.235"
  },
  "nonAdjacent": {
  }
}
```


## Correctly Handle Configuration Files
As this is a multi-layer distributed system, a few configuration files are used to facilitate the deployment for different nodes (or sites).  They need to be carefully checked before running/testing the system.

#### kms.conf

Sample applications provided (qTox and tls-demo) require kms.conf to be present under
the $HOME/.qkd directory.

This file can be manually created by puting following entries:
```
http://localhost:9992/uaa/oauth/token
http://localhost:8095/api/newkey
http://localhost:8095/api/getkey
C
```

Depending on where the KMS node is deployed, `localhost` can be replaced by the IP address of
the host running the KMS layer.
The last line represents the KMS site id, *which should be updated with the
correct site id*. Above is just an example.

##### config-repo

config-repo directory under `$HOME/` contains all the configuration property
files required by all the microservices.

All the files reside under <top level directory>/qkd-net/kms/config-repo from where
they are copied to `$HOME/config-repo` and checked in a local git repository.

##### site.properties

This file is located under <top level directory>/qkd-net/kms/kms-service/src/main/resources/.

Explanation of site specific configuration properties used by KMS service:
```C
//Number of keys per block
kms.keys.blocksize=1024
//Size of a key in bytes
kms.keys.bytesize=32
//Top level location for locating key blocks
kms.keys.dir=${HOME}/.qkd/kms/pools/
//IP address of the host where key routing service is running
qnl.ip=localhost
//Port on which key routing service is listening for key block requests
qnl.port=9292
```

##### config.yaml (KMS QNL Service)


This file is copied from <top level directory>/qkd-net/kms/kms-qnl-service/src/main/resources/
to $HOME/.qkd/kms/qnl from where it is used by KMS QNL service


Expalantion of configuration properties used by KMS QNL service:

```Yaml
//Port on which KMS QNL service is listening
port: 9393
//Size of a key in bytes
keyByteSz: 32
//Number of keys in a block. Key routing service provides KMS keys in blocks of size    
keyBlockSz: 1024
//Location where KMS QNL service puts the pushed key blocks from key-routing service
poolLoc: kms/pools
```
##### config.yaml and routes.json (Key-Routing Service)

###### routes.json:

This file is copied from <top level directory>/qkd-net/qnl/conf/ to
`$HOME/.qkd/qnl`.

Topology information is contained in `route.json`.
This information is populated manually for adjacent (neighboring) nodes, while the non-adjacent nodes will be discovered by the routing module. Adjacent nodes section contains the name of the node as key and it's IP address as the value.

Example `route.json` file for a QNL Node "B":
```json
{
  "adjacent": {
    "A": "192.168.1.101",
    "C": "192.168.1.103"
  },
  "nonAdjacent": {
  }
}
```

###### config.yaml:

This file is copied from <top level directory>/qkd-net/qnl/conf/ to
$HOME/.qkd/qnl.  It contains various configuration parameters for the key routing service
running as a network layer.

Brief explanation of various properties is given with the following example file:

```yaml
//Base location for finding other paths and configuration files.
base: .qkd/qnl
//Route configuration file name.
routeConfigLoc: routes.json
//Location where QLL puts the key blocks for QNL to carve out key blocks for KMS.
qnlSiteKeyLoc: qll/keys
//Each site has an identifier uniquely identifying that site.
siteId: A
//Port on which key routing service is listening for key block requests.
port: 9292
//Size of a key in bytes.
keyBytesSz: 32
//Number of keys in a block. Key routing service provides KMS keys in blocks of size.  
keyBlockSz: 1024
//Key routing service expects QLL to provide keys in blocks of qllBlockSz.
qllBlockSz: 4096
//IP address of the host running KMS QNL service.
kmsIP: localhost
//Port on which KMS QNL service is listening.
kmsPort: 9393
//One Time Key configuration.
OTPConfig:
 //Same as above
 keyBlockSz: 1024
 //Location of the OTP key block.
 keyLoc: otp/keys
```

## Build and Install Services

For building the services,

```shell
cd <top level director>/qkd-net/kms
./scripts/build
```

For running the services

```
cd <top level director>/qkd-net/kms
./scripts/run
```

### Checking registration service

 Open a browser, check http://localhost:8761/

### Testing KMS service

Since OAuth is enabled, first access token is fetched:

```shell
curl -X POST -H"authorization:Basic aHRtbDU6cGFzc3dvcmQ=" -F"password=bot" -F"client_secret=password" -F"client=html5" -F"username=pwebb" -F"grant_type=password" -F"scope=openid"  http://localhost:9992/uaa/oauth/token | jq
```

Sample output:
```json
{
  "access_token": "291a94d5-d624-4269-a2bb-07db62130bb3",
  "token_type": "bearer",
  "refresh_token": "99d4cdd9-641e-4290-ac4b-865b1d7068a6",
  "expires_in": 43017,
  "scope": "openid"
}
```

Once the access token is available, different endpoints can be accessed.

Assuming we have 2 sites setup with siteid as A and B repsectively.
Some application from site A initiates a connection to an application on
site B.
Pre-shared key fetching can be simulated using the two calls below.  


There are two REST API endpoints:

1. New Key

Application at site B makes a *newkey* call.

**Request**

  Method :      POST
  URL path :    /api/newkey
  URL params:   siteid=[alhpanumeric] e.g. siteid=A

**Response**
Format in JSON
```
{
  index: Index of he key
  hexKey: Key in hexadecimal format
  blockId: Id of the block containing the key
}
```
e.g. request-response:

```shell
curl 'http://localhost:8095/api/newkey?siteid=A' -H"Authorization: Bearer 291a94d5-d624-4269-a2bb-07db62130bb3" | jq
```

```json
{
  "index": 28,
  "hexKey": "a2e1ff3429ff841f5d469893b9c28cbcb586d55c8ecf98c83c704824e889fc43"
  "blockId": "133ad3f8de6cc9da"
}
```

2. Get Key

Application on site A makes the *getkey* call by using part of the response from the
newkey call above.

NOTE:
Since the sites are different hence OAuth tokens will be different, which
means a new OAuth token like above is fetched first by the application on site A.

**Request**
```
  Method :      POST
  URL path :    /api/getkey
  URL params:   siteid=[alhpanumeric] e.g. siteid=B
                blockid=
                index=[Integer]
```

***Response***
  Format in JSON
```
  {
    index: Index of he key
    hexKey: Key in hexadecimal format
    blockId: Id of the block containing the key
  }
```

e.g. request-response:

```shell
curl 'http://localhost:8095/api/getkey?siteid=B&index=1&blockid=' -H"Authorization: Bearer abcdef12-d624-4269-a2bb-07db62130bb3" | jq
```

```json
{
  "index": 1,
  "hexKey": "4544a432fb045f4940e1e2fe005470e1a35d85ede55f78927ca80f46a0a4b045"
  "blocId": "8a94dc22e761de8c5501addc05"
}
```

TEAM
----

The Open QKD Network project is led by Professors [Michele Mosca](http://faculty.iqc.uwaterloo.ca/mmosca/) and [Norbert Lutkenhaus](http://services.iqc.uwaterloo.ca/people/profile/nlutkenh) at Institute for Quantum Computing (IQC) of the University of Waterloo.

### Contributors

Main contributors to this master branch so far include:
- Shravan Mishra (University of Waterloo)
- Kaiduan Xie (University of Waterloo)
- Dr. Xinhua Ling (University of Waterloo, evolutionQ Inc.)
