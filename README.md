# Running Minecraft in Kubernetes with K3s

## Running K3s in Docker
To run K3s in Docker you can use the helper utillity K3d, this can be installed from [https://k3d.io/#installation](https://k3d.io/#installation)

Make sure Docker is running and then run the following command to create a single node Kubernetes cluster called `minecraft` and expose the port 25565 on localhost to the cluster port 30001.

```shell
k3d create cluster minecraft -p 25565:30001
```

## Running Minecraft in Kubernetes

To run Minecraft application you can configure a deployment like the below example. This runs a single insance of the Minecraft Docker container and configures the application through environment variables.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: minecraft-deployment
  labels:
    app: minecraft
spec:
  replicas: 1
  selector:
    matchLabels:
      app: minecraft
  template:
    metadata:
      labels:
        app: minecraft
    spec:
      containers:
      - name: minecraft
        image: hashicraft/minecraft:v1.12.2
        ports:
        - containerPort: 25565
        env:
        - name: WORLD_BACKUP
          value: https://github.com/nicholasjackson/hashicraft/releases/download/v0.0.0/world2.tar.gz
        - name: WHITELIST_ENABLED
          value: "false"
```

## Exposing the Minecraft application

To expose the minecraft application on the cluster you can create a `NodePort` service. The port exposed on the node is the same port that we configured when setting up `K3d`.

```yaml
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: minecraft
  name: minecraft
spec:
  ports:
  - name: minecraft
    nodePort: 30001
    port: 25565
    protocol: TCP
    targetPort: 25565
  selector:
    app: minecraft
  type: NodePort
```

