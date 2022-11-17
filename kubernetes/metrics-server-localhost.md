# Metrics Server Local Host

https://dev.to/docker/enable-kubernetes-metrics-server-on-docker-desktop-5434

Metrics server isn't included with Docker Desktop's installation of [[kubernetes]] and to install it we will have to download the latest `components.yaml` file from [Metrics-Server](https://github.com/kubernetes-sigs/metrics-server/releases) releases page and open it in your text editor. 

Add the line --kubelet-insecure-tls under the args section as shown below