---
layout: default
title: Components
nav_order: 1
parent: Installation
---

Direktiv consists of three components where the secrets service is optional. All components share the same configuration file but not all avlues are required for all components.

DIAGRAM HOW THEY INTERACT

### **Ingress/Flow Service**

This component handles API requests and manages the flow logic of the workflows in direktiv.

*Starting flow service:*
```
./direktiv -t=w -c /my.conf
```

Available configuration:

```toml
[Database]
  DB = "host=127.0.0.1 port=5432 user=postgres password=example sslmode=disable"
  
[flowAPI]
  Bind = ":7777"
  Endpoint = "localhost:7777"

[ingressAPI]
  Bind = ":9999"
  Endpoint = "localhost:9999"

[isolateAPI]
  Endpoint = "localhost:9999"
```

### **Action Service**

The action service is responsible to execute containers either as isolates (VM) or containers. The decision to run the workflow actions as either isolates or containers depends on the isolation requirements, e.g. on a shared platform it might be better to use virtual machine isolation and provide only limited disk space for each isolate.

If the isolation level is set to 'vorteil' the workflow actions are getting executed as firecracker virtual machines. To do so they will be converted from containers to vorteil virtual machines.

The first time the action is getting called in a workflow it takes time to convert the container to a virtual disk. To avoid running this conversion every time there are two levels of caching.

The first level is based on [Minio](https://min.io/). Because direktiv needs S3 storage this can be either us#### ed as S3 backend or as an API layer to e.g. Google Cloud Storage depending on the configuration of Minio.

DIAGRAM HERE MINIO VS GCP

On a successful conversion the disk is getting stored as an encrypted blob via [Minio](https://min.io/) so other nodes in the cluster can download the disk if required. Additionally the disk is stored locally for instant access. The local cache uses 70 percent of the filesystem.

*Starting isolate service:*
```
./direktiv -t=i -c /my.conf
```

Available configuration:

```toml
[Database]
  DB = "host=127.0.0.1 port=5432 user=postgres password=example sslmode=disable"

[InstanceLogging]
  Driver = "database"

[isolateAPI]
  Bind = ":9999"
  Isolation = "vorteil" # or "container"

[Minio] # Only required if isolation level is 'vorteil'
  Encrypt = "encryptit"
  Endpoint = "localhost:9000"
  User = "example-user"
  Password = "example-password"
  Region = ""
  SSL = 0
  Secure = 0

[flowAPI]
  Endpoint = "localhost:7777"

[Kernel]
  Linux = "21.3.2" # vorteil linux kernel version
  Runtime = "21.3.2"

[flowAPI]
   # used as a default login if none is provided in the namespace via secrets
  [flowAPI.Registry]
  Name = "docker.io"
  Token = "TH1S-1SNT-4-R34L-T0K3N"
  User = "my-user"
```


### **Secrets Service (optional)**

Easy way to store and provide secrets to direktiv workflows. Secrets are getting stored per namespace and the key and encrypted values are physically seperated.

*Starting secrets service:*
```
./direktiv -t=s -c /my.conf
```

Available configuration:

```toml
[secretsAPI]
  Bind = ":2610"
  DB = "host=127.0.0.1 port=5432 user=postgres password=example sslmode=disable"
```
