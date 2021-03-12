---
layout: default
title: Configuration
nav_order: 40
---

# Configuration

The Direktiv server can be provided a configuration file on startup (`-c /conf.toml`) that will be used to configure the various server components. Below is an example of a server configuration file, along with a brief description of what is expected of each field.

The following flags instruct the server to run certain components (any combination of the three flags may be provided):

```
-i    Enables the 'Isolates' server component
-w    Enables the 'Workflow' server component
-s    Enables the 'Secrets' server component
```

```toml
[Certs]
  Directory = "/tmp/certs"
  Secure = 0

[Database]
  DB = "host=127.0.0.1 port=5432 user=postgres password=example sslmode=disable"

[InstanceLogging]
  Driver = "database"

[Kernel]
  Linux = " "
  Runtime = " "

[Minio]
  Encrypt = " "
  Endpoint = "localhost:9000"
  Password = "example-password"
  Region = " "
  SSL = 0
  Secure = 0
  User = "example-user"

[flowAPI]
  Bind = ":7777"
  Endpoint = "localhost:7777"

  [flowAPI.Registry]
    Name = "docker.io"
    Token = "TH1S-1SNT-4-R34L-T0K3N"
    User = "my-user"

[healthAPI]
  Bind = ":9999"
  Endpoint = "localhost:9999"

[ingressAPI]
  Bind = ":9999"
  Endpoint = "localhost:9999"

[isolateAPI]
  Bind = ":9999"
  Endpoint = "localhost:9999"
  Isolation = "vorteil" # or "container"

[secretsAPI]
  Bind = ":9999"
  DB = "host=127.0.0.1 port=5432 user=postgres password=example sslmode=disable"
  Endpoint = "localhost:9999"

```

## Fields

### Certs
- Directory
  - Points to an existing directory containing TLS certificate/key files for direktiv server components to establish a secure server/client. Within the directory, an individual folder is expected for each of the following four direktiv server components ('flow', 'ingress', 'isolate', 'secrets').
  - Files within each component folder are expected to be named:
    - ca.cert
    - ca.key
    - ca.pem
    - ca.srl
    - client.key
    - client.pem
    - client.req
    - server.key
    - server.pem
- Secure
  - Setting to `1` will require that all certs are verifiable. To skip certificate verification, set this field to `0`.

### Database
  - DB
    - Contains a database 'connection string', used for the direktiv server to establish a connection to a postgres server.

### InstanceLogging
  - Driver
    - Currently only supports `database`, which will store logs from instances in the configured database. More drivers will be added over time.

### Kernel
  - Linux
    - Specifies the version of the kernel to use.
  - Runtime
    - Specifies the version of the kernel used for firecracker instances (usually the same value as the `linux` field).

### Minio
  - Encrypt
    - An encryption key used for encrypting/decrypting files.
  - Endpoint
    - Endpoint for Minio API requests to be sent.
  - User
    - Username for authentication/authorisation purposes.
  - Password
    - Password for authentication/authorisation purposes.
  - Region
    - Specifies a region if using Minio with a remote cloud provider (such as Google Cloud Platform or Amazon Web Services).
  - SSL
    - Specifies whether a secure connection (HTTPS) should be used. Setting this field to `0` will result in the use of HTTP.
  - Secure
    - Supported values: `0`, `1`. If set to `0`, certificates will not be verified when communicating with the Minio server.

### flowAPI
  - Bind
    - Address to bind the flowAPI endpoint running on the direktiv server.
  - Endpoint
    - Endpoint to send flowAPI requests to.
  - Registry
    - Name
      - Name of the targeted container registry (ie. `docker.io`).
    - User
      - Username that will be used when authenticating/authorising with the container registry.
    - Token
      - Authentication token for the specified user.

### healthAPI
  - Bind
    - Address to bind the healthAPI endpoint running on the direktiv server.
  - Endpoint
    - Endpoint to send healthAPI requests to.

### ingressAPI
  - Bind
    - Address to bind the ingressAPI endpoint running on the direktiv server.
  - Endpoint
    - Endpoint to send ingressAPI requests to.

### isolateAPI
  - Bind
    - Address to bind the isolateAPI endpoint running on the direktiv server.
  - Endpoint
    - Endpoint to send isolateAPI requests to.
  - Isolation
    - Accepts 'vorteil' (runs actions in Vorteil virtual machines) or 'container' (runs actions in containers)

### secretsAPI
  - Bind
    - Address to bind the secretsAPI endpoint running on the direktiv server.
  - Endpoint
    - Endpoint to send secretsAPI requests to.
  - DB
    - Contains a database 'connection string', used for the direktiv server to establish a connection to a postgres server.

## Environment Variable Overrides

A number of environment variables can be provided which will be used to set/override the value of a field in a configuration file. Below is a table containing all supported 'override' environment variables.

| KEY | FIELD |
|---|---|
| DIREKTIV_FLOW_BIND | flowAPI.Bind |
| DIREKTIV_FLOW_ENDPOINT | flowAPI.Endpoint |
| DIREKTIV_FLOW_REGISTRY | flowAPI.Registry.Name |
| DIREKTIV_FLOW_REGISTRY_USER | flowAPI.Registry.User |
| DIREKTIV_FLOW_REGISTRY_TOKEN | flowAPI.Registry.Token |
| DIREKTIV_HEALTH_BIND | healthAPI.Bind |
| DIREKTIV_HEALTH_ENDPOINT | healthAPI.Endpoint |
| DIREKTIV_INGRESS_BIND | ingressAPI.Bind |
| DIREKTIV_INGRESS_ENDPOINT | ingressAPI.Endpoint |
| DIREKTIV_ISOLATE_BIND | isolateAPI.Bind |
| DIREKTIV_ISOLATE_ENDPOINT | isolateAPI.Endpoint |
| DIREKTIV_SECRETS_BIND | secretsAPI.Bind |
| DIREKTIV_SECRETS_ENDPOINT | secretsAPI.Endpoint |
| DIREKTIV_SECRETS_DB | secretsAPI.DB |
| DIREKTIV_DB | database.DB |
| DIREKTIV_MINIO_ENDPOINT | Minio.Endpoint |
| DIREKTIV_MINIO_USER | Minio.User |
| DIREKTIV_MINIO_PASSWORD | Minio.Password |
| DIREKTIV_MINIO_INSECURE | Minio.Insecure |
| DIREKTIV_MINIO_SSL | Minio.SSL |
| DIREKTIV_MINIO_ENCRYPT | Minio.Encrypt |
| DIREKTIV_MINIO_REGION | Minio.Region |
| DIREKTIV_KERNEL_LINUX | Kernel.Linux |
| DIREKTIV_KERNEL_RUNTIME | Kernel.Runtime |
| DIREKTIV_INSTANCE_LOGGING_DRIVER | InstanceLogging.Driver |
