---
layout: default
title: Kubernetes
nav_order: 2
parent: Installation
---

At this stage we are providing an example installation process for kubernetes. We are contantly trying to improve it and are hoping to provide helm charts in the near future. In the meantime our demo installation YAML files can be modified to create a production ready deployment. To run the demo setup the installation has four components.

**WARNING:** Installation YAMLs for testing only!

### Support Service (Minio, Postgres)

```YAML
apiVersion: v1
kind: List
items:
- apiVersion: v1
  kind: Service
  metadata:
    name: direktiv-support
    labels:
      run: direktiv-support
  spec:
    ports:
    - port: 9000
      name: minio
      protocol: TCP
    - port: 5432
      name: postgresql
      protocol: TCP
    selector:
      run: direktiv-support
- apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: direktiv-support
  spec:
    selector:
      matchLabels:
        run:  direktiv-support
    replicas: 1
    template:
      metadata:
        labels:
          run:  direktiv-support
      spec:
        containers:
        - name: minio
          image: minio/minio
          command: ["minio"]
          args: ["server", "/data"]
          env:
          - name: MINIO_ACCESS_KEY
            value: "vorteil"
          - name: MINIO_SECRET_KEY
            value: "vorteilvorteil"
          ports:
            - containerPort: 9000
        - name: postgres
          image: postgres
          env:
          - name: POSTGRES_USER
            value: "vorteil"
          - name: POSTGRES_PASSWORD
            value: "vorteilvorteil"
          ports:
            - containerPort: 5432
```

### Isolate Service (Running containers / isolates)

```YAML
apiVersion: v1
kind: List
items:
- apiVersion: v1
  kind: Service
  metadata:
    name: direktiv-isolate
    labels:
      run: direktiv-isolate
  spec:
    ports:
    - port: 8888
      name: isolate
      protocol: TCP
    selector:
      run: direktiv-isolate
- apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: direktiv-isolate
  spec:
    selector:
      matchLabels:
        run: direktiv-isolate
    replicas: 1
    template:
      metadata:
        labels:
          run: direktiv-isolate
      spec:
        containers:
        - name: isolate
          image: gerke74/isolate
          securityContext:
            privileged: true
          env:
          - name: DIREKTIV_ISOLATE_BIND
            value: "0.0.0.0:8888"
          - name: DIREKTIV_DB
            value: "host=direktiv-support port=5432 user=vorteil dbname=direktiv password=vorteilvorteil sslmode=disable"
          - name: DIREKTIV_MINIO_ENDPOINT
            value: "direktiv-support:9000"
          - name: DIREKTIV_ISOLATION
            value: "container"
          ports:
            - containerPort: 8888
```


FLOW

```YAML
apiVersion: v1
kind: List
items:
- apiVersion: v1
  kind: Service
  metadata:
    name: direktiv-flow
    labels:
      run: direktiv-flow
  spec:
    ports:
    - port: 6666
      name: ingress
      protocol: TCP
    - port: 7777
      name: flow
      protocol: TCP
    selector:
      run: direktiv-flow
- apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: direktiv-flow
  spec:
    selector:
      matchLabels:
        run: direktiv-flow
    replicas: 1
    template:
      metadata:
        labels:
          run: direktiv-flow
      spec:
        containers:
        - name: flow
          image: gerke74/flow
          securityContext:
            privileged: true
          env:
          - name: DIREKTIV_INGRESS_BIND
            value: "0.0.0.0:6666"
          - name: DIREKTIV_FLOW_BIND
            value: "0.0.0.0:7777"
          - name: DIREKTIV_DB
            value: "host=direktiv-support port=5432 user=vorteil dbname=direktiv password=vorteilvorteil sslmode=disable"
          - name: DIREKTIV_ISOLATE_ENDPOINT
            value: "direktiv-isolate:8888"
          - name: DIREKTIV_SECRETS_ENDPOINT
            value: "direktiv-secrets:2610"            
          ports:
            - containerPort: 6666
              name: ingress
            - containerPort: 7777
              name: flow

```

secrets

```
apiVersion: v1
kind: List
items:
- apiVersion: v1
  kind: Service
  metadata:
    name: direktiv-secrets
    labels:
      run: direktiv-secrets
  spec:
    ports:
    - port: 2610
      name: isolate
      protocol: TCP
    selector:
      run: direktiv-secrets
- apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: direktiv-secrets
  spec:
    selector:
      matchLabels:
        run: direktiv-secrets
    replicas: 1
    template:
      metadata:
        labels:
          run: direktiv-secrets
      spec:
        containers:
        - name: secrets
          image: gerke74/secrets
          securityContext:
            privileged: true
          env:
          - name: DIREKTIV_SECRETS_BIND
            value: "0.0.0.0:2610"
          - name: DIREKTIV_SECRETS_DB
            value: "host=direktiv-support port=5432 user=vorteil dbname=direktiv password=vorteilvorteil sslmode=disable"
          ports:
            - containerPort: 2610
            ```
