# KubeService

A simple CRD controller for micro-service management.

![](https://github.com/Coderhypo/KubeService/blob/master/docs/img/KubeService.gif?raw=true)

## What's KubeService

KubeService is a CRD controller build on KubeBuilder, 
and this define some CR in kubernetes like: `App`、`MicroService` for micro-service management easier .

The logical structure like:

![](https://github.com/Coderhypo/KubeService/blob/master/docs/img/logical_structure.jpg?raw=true)

## Feature

### Deploy management

User can define a `App` resource to manage multiple micro services, 
and use DeployVersion to making multiple versions coexist.

### Load balancing configuration

User use define `CurrentVersion` make the current version provide services, 
and define `Canary` to Canary Deploy other versions.

![](https://github.com/Coderhypo/KubeService/blob/master/docs/img/loadbalance.jpg?raw=true)

## Usage

### install and run

Add CRD to kubernetes:

```bash
make install
```

run controller on local

```bash
make run
```

### App Demo

```yaml
apiVersion: app.o0w0o.cn/v1
kind: App
metadata:
  name: voting-sample
spec:
  microServices:
    - name: voting-web
      spec:
        loadBalance:
          service:
            name: voting-web
            spec:
              ports:
                - protocol: TCP
                  port: 80
                  targetPort: 80
          ingress:
            name: voting-web
            spec:
              rules:
                - host: voting.o0w0o.cn
                  http:
                    paths:
                      - path: /
                        backend:
                          serviceName: voting-web
                          servicePort: 80
        versions:
          - name: v1
            template:
              replicas: 2
              selector:
                matchLabels:
                  app: voting-web
              template:
                metadata:
                  labels:
                    app: voting-web
                spec:
                  containers:
                    - image: daocloud.io/w0v0w/voting-demo-voting:v1
                      name: voting-web
          - name: v2
            canary:
              weight: 30
            template:
              replicas: 1
              selector:
                matchLabels:
                  app: voting-web-for-kid
              template:
                metadata:
                  labels:
                    app: voting-web-for-kid
                spec:
                  containers:
                    - image: daocloud.io/w0v0w/voting-demo-voting:v2
                      name: voting-web-for-kid
        currentVersionName: v1
    - name: voting-result
      spec:
        loadBalance:
          service:
            name: voting-result
            spec:
              ports:
                - protocol: TCP
                  port: 80
                  targetPort: 80
          ingress:
            name: voting-result
            spec:
              rules:
                - host: result.voting.o0w0o.cn
                  http:
                    paths:
                      - path: /
                        backend:
                          serviceName: voting-result
                          servicePort: 80
        versions:
          - name: v1
            template:
              replicas: 1
              selector:
                matchLabels:
                  app: voting-result
              template:
                metadata:
                  labels:
                    app: voting-result
                spec:
                  containers:
                    - image: daocloud.io/w0v0w/voting-demo-result:v1
                      name: voting-result
        currentVersionName: v1

---

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: redis
  name: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
        - image: redis
          name: redis

---

apiVersion: v1
kind: Service
metadata:
  labels:
    app: redis
  name: redis
spec:
  ports:
    - port: 6379
      protocol: TCP
      targetPort: 6379
  selector:
    app: redis
```
