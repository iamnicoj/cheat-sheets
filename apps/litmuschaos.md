# LitmusChaos

# Litmus Quickstart


>Note: Much of this is taken from the [ChaosCenter Cluster Scope Installation] doc.

[ChaosCenter Cluster Scope Installation]: https://docs.litmuschaos.io/docs/getting-started/installation/
 

## Installation

You can install this on either a local (e.g. Docker Desktop) or remote kubernetes cluster (e.g. AKS). In either case, to install:
 
>Note: Only choose one of the following methods.

* Using Helm
  
```sh

# Add to Helm

helm repo add litmuschaos https://litmuschaos.github.io/litmus-helm/

  

# Separate out Litmus-specific resources into its own namespace

kubectl create ns litmus

  

# Install the helm chart onto the cluster

# If the cluster is local, you can append "--set portal.frontend.service.type=NodePort"

# example versions: 2.13.0, 2.14.0

helm install chaos litmuschaos/litmus --namespace litmus --version <YOUR_LITMUS_VERSION>

```

* OR using `kubectl`

```sh

# Separate out Litmus-specific resources into its own namespace

kubectl create ns litmus

  

# Apply a specific release YAML of Litmus

# e.g. kubectl apply -f https://litmuschaos.github.io/litmus/2.14.0/litmus-2.14.0.yaml -n litmus

kubectl apply -f https://litmuschaos.github.io/litmus/<YOUR_LITMUS_VERSION>/litmus-<YOUR_LITMUS_VERSION>.yaml -n litmus

```

## Accessing the Litmus UI

Instructions for accessing the Litmus ChaosCenter installed above will depend slightly depending on where Litmus is installed and how it's been exposed:

* **NodePort**

If Litmus was exposed via `NodePort`, you should be able to access it from your browser at `<NODEIP>:<PORT>`. This also works with a LoadBalancer in the same way: `<LoadBalancerIP>:<PORT>`.


* **Port-forwarding**

  
If Litmus is installed on a remote cluster (e.g. AKS), you can also just port-forward to access the UI locally:

 ```sh

# The UI will then be accessible at 127.0.0.1:9091

kubectl port-forward svc/chaos-litmus-frontend-service -n litmus 9091:9091

```

In either case, once accessed, the default Litmus credentials (as stated in the [docs]) are:

```

Username: admin

Password: litmus

```

[docs]: https://docs.litmuschaos.io/docs/getting-started/installation/#accessing-the-chaoscenter


## Install Chaos Experiments

After you've accessed the Litmus UI, but before you've run your first simulated workload, you must install the standard set of chaos experiments for your given version of litmus. See the following:  

```sh

kubectl apply -f https://hub.litmuschaos.io/api/chaos/<YOUR_LITMUS_VERSION>?file=charts/generic/experiments.yaml -n litmus

```

  
## Running this simulated workload


There are two methods of running this simulated workload; via Litmus Chaos or via Argo Workflows (as they share Workflow YAML schemas). However, the preferred method of running this is via Litmus Chaos.

If you have any issues running the workload related to service accounts or permissions, try:
 

```sh

kubectl apply -f rbac.yaml

```


>Note: make sure you have also already applied any required `WorkflowTemplate`'s before running a scenario! For instance:


>`kubectl apply -f chaos-templates\chaos-templates-distribution.yaml`
>`kubectl apply -f chaos-templates\chaos-templates-behaviors.yaml`
>`kubectl apply -f chaos-templates\chaos-templates-deployment.yaml`
>`kubectl apply -f chaos-templates\chaos-templates-distribution.yaml`

  

### To run the simulated workload with Litmus Chaos:

 In the Litmus Chaos UI, click "Schedule a Chaos Scenario":
  

![Run with Litmus Chaos](/assets/litmus-launch-scenario.png)

  
Upload the `simulated-workload-poc.yaml` Chaos Scenario and watch it run in the UI. Also it's recommended to check on the pods in both the "default" and "litmus" namespaces as it runs.

### To run the simulated workload with Argo Workflows

 First ensure you have [Argo Workflows installed on your cluster] and that you have the [argo CLI tool installed]. Then run:

  [Argo Workflows installed on your cluster]: https://argoproj.github.io/argo-workflows/quick-start/#install-argo-workflows

[argo CLI tool installed]: https://argoproj.github.io/argo-workflows/quick-start/#install-the-argo-workflows-cli

  ```sh

argo submit --watch ./simulated-workload-poc.yaml

```

  As this runs, you'll be able to follow along both in the terminal as well as in the Argo WF UI.

# Manual env reset

## litmusctl installation
 
To install the `litmusctl` follow [this](https://github.com/litmuschaos/litmusctl) instructions. So far we have been using the `0..13.0` and `0.14.0` versions using the `2.13` and `2.14` backend.

You can also follow basic instructions on how to use the `litmusctl` [here](https://github.com/litmuschaos/litmusctl/blob/master/Usage.md)
 
## manual script example

The following scripts helps cleaning and resetting the litmus scenario run and the target apps namespace


```bash

# check if port is correct

litmusctl config set-account --endpoint="http://127.0.0.1:9195/" --username="admin" --password="litmus"

  

# litmusctl get projects

  

DEFAULT_PROJECT=$(litmusctl get projects | awk '$1 ~ /[({]?[a-fA-F0-9]{8}[-]?([a-fA-F0-9]{4}[-]?){3}[a-fA-F0-9]{12}[})]?/' | awk '{ print $1 }' | awk 'NR>1{exit};1')

  

# print the available chaos-scenaios - We should filter those that are running only maybe

litmusctl get chaos-scenarios --project-id=$DEFAULT_PROJECT

  

# this next line replaces LAST_SCENARIO with latest scenario id query on previews command

LAST_SCENARIO=$(litmusctl get chaos-scenarios --project-id=$DEFAULT_PROJECT | awk '$1 ~ /[({]?[a-fA-F0-9]{8}[-]?([a-fA-F0-9]{4}[-]?){3}[a-fA-F0-9]{12}[})]?/' | awk '{ print $1 }' | awk 'NR>1{exit};1')


# delete the latest scenario defined in LAST_SCENARIO

litmusctl delete chaos-scenario $LAST_SCENARIO --project-id=$DEFAULT_PROJECT

  

# delete all other ephemeral components

k delete chaosengine --all -n litmus

k delete jobs --all -n litmus

k delete ns nico --force --grace-period=0

k create ns nico

k delete pod --field-selector=status.phase==Succeeded -n litmus

  
k apply -f ./chaos-templates -n litmus

# When trying to exectute an scenario from a yaml file
CHAOS_DELEGATE=litmusctl get chaos-delegates --project-id=$DEFAULT_PROJECT

litmusctl create chaos-scenario -f custom-chaos-scenario.yml --project-id=$DEFAULT_PROJECT --chaos-delegate-id=""

```

  

## manually change labels while the scenario is running

  

if you want to manually play with labels to be attach to stressors apply different set of labels every few minutes. Update the namespace and labels based on your own testing.

  

```bash

kubectl label pods -l workloadId=small-interactive-ads -n nico --overwrite stressor=60cpu

kubectl label pods -l workloadId=medium-interactive-ads -n nico --overwrite stressor=20cpu

kubectl label pods -l workloadId=large-interactive-ads -n nico --overwrite stressor=20cpu

kubectl label pods -l workloadId=x-large-interactive-ads -n nico --overwrite stressor=20cpu

```

  

TODO: use something like kustomize to dynamically update image paths to harbor repository on

  

> 2022/11/17 16:25:37 Error Creating Resource : ChaosEngine.litmuschaos.io 'pod-cpu-engine-smxqr' is invalid: spec.appinfo.appkind: Invalid value: 'pod': spec.appinfo.appkind in body should match '^(^$|deployment|statefulset|daemonset|deploymentconfig|rollout)$'

  

## Known issues

  

Know issue getting this error in docker desktop:

>Lab 4.6 Install crictl : issue with starting crio.service

  

Make sure you configure the `container_runtime` and the `socket_path` to run with the DD compliant framework

  

Anyway there will be issues running the cgroup error that I haven't figured it out just yet.

  

Additionally, running the scenario that deployes dozens of pods on small cluster can cause race conditions where stressor hogs fails randomly.

