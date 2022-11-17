# kustomize

A configuration manager for ([[kubernetes]])

Project Homepage: [kustomize.io](https://kustomize.io/)


## apply with `Kustomize files`

Though Apply can be run directly against Resource Config files or directories using `-f`, it is recommended to run Apply against a `kustomization.yaml` using `-k`. The `kustomization.yaml` allows users to define configuration that cuts across many Resources (e.g. namespace).

``` YAML
# kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# list of Resource Config to be Applied
resources:
- deployment.yaml

# namespace to deploy all Resources to
namespace: default

# labels added to all Resources
commonLabels:
  app: example
  env: test

```

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: the-deployment
spec:
  replicas: 5
  template:
    containers:
      - name: the-container
        image: registry/container:latest
```

Users run Apply on directories containing `kustomization.yaml` files using `-k` or on raw ResourceConfig files using `-f`.

```bash
# Apply the Resource Config
kubectl apply -k .

# View the Resources
kubectl get -k .
```

# [Container Images](https://kubectl.docs.kubernetes.io/guides/config_management/container_images/#container-images)

Consider the following `deployment.yaml` file,

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: the-deployment
spec:
  template:
    spec:
      containers:
      - name: mypostgresdb
        image: postgres:8
      - name: nginxapp
        image: nginx:1.7.9
      - name: myapp
        image: my-demo-app:latest
      - name: alpine-app
        image: alpine:3.7
```

```yaml
# kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

images:
- name: postgres
  newName: my-registry/my-postgres
  newTag: v1
- name: nginx
  newTag: 1.8.0
- name: my-demo-app
  newName: my-app
- name: alpine
  digest: sha256:24a0c4b4a4c0eb97a1aabb8e29f18e917d05abfe1b7a7c07857230879ce7d3d3

resources:
- deployment.yaml
```




