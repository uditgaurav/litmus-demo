# Litmus Kubernetes Demo Environment

The purpose of this repository is to familiarize oneself with running litmus chaos experiments in a realistic app environment running multiple services on a Kubernetes cluster. This repo is forked from the [zebrium kubernetes demo](https://github.com/zebrium/zebrium-kubernetes-demo) which is very similar to what is detailed here plus the setup of the zebrium automated incident detection framework. 

It makes it easy to spin up a fully deployed [GKE](https://cloud.google.com/kubernetes-engine/) cluster with a microservice application 
[Sock Shop](https://github.com/microservices-demo/microservices-demo), and 
[Litmus Chaos Engine](https://litmuschaos.io/) to create chaos scenarios.

For more background, please read [Using Autonomous Monitoring with Litmus Chaos Engine on Kubernetes](https://www.zebrium.com/blog/using-autonomous-monitoring-with-litmus-chaos-engine-on-kubernetes?utm_campaign=Chaos&utm_source=chaos&utm_medium=github).

![Deployed Services](https://www.zebrium.com/hs-fs/hubfs/Zebrium%20and%20Litmus%20Chaos%20Engine%20components.png?width=1461&name=Zebrium%20and%20Litmus%20Chaos%20Engine%20components.png)

After cloning this repository, installing the requirements listed below, and using the `start` command to create the fully deployed cluster, you will be able to run Litmus Chaos experiments using the `test` command in the cluster. You can find all the experiment configuration under the `/litmus` directory of this repository and the script to deploy and run them in `manage.py`.

It currently only works with GKE so you will need a Google Cloud account to run this environment, but support for Amazon and Azure is planned in future.

## Requirements

1. Python 3.7 or above
2. Python Dependencies: `pip install -r requirements.txt`
3. Google Cloud Login: https://console.cloud.google.com/
4. GCloud CLI installed locally and logged in: https://cloud.google.com/sdk/docs/quickstarts
5. Kubectl installed locally: https://kubernetes.io/docs/tasks/tools/install-kubectl/
6. Helm installed locally: https://helm.sh/docs/intro/install/


## Usage

To see full command line options use the `-h` flag:

```bash
./manage.py -h
```

This will output the following:

```bash
usage: manage.py [-h] {start,test,stop} ...

Spin up Litmus Demo Environment on Kubernetes.

positional arguments:
  {start,test,list,stop}
    start               Start a GKE Cluster with demo environment
                        deployed.
    test                Run Litmus ChaosEngine Experiments inside
                        demo environment.
    list                List all available Litmus ChaosEngine Experiments
                        available to run.
    stop                Shutdown the GKE Cluster with demo
                        environment deployed.
```

## Startup

To start the GKE cluster and deploy all the required components:

```bash
./manage.py start --project {GC_PROJECT} --key {ZE_KEY}
```

## Test

To run all the Litmus ChaosEngine experiments:

```bash
./manage.py test
```
You can optionaly add the `--wait=` argument to change the wait time between experiments in minutes. By default
it is 1 min.

To run a specific experiment (found under the ./litmus directory):

```bash
./manage.py test --test=container-kill
```

### Notes

- To view application deployment picked, success/failure of reconcile operations (i.e., creation of chaos-runner pod or lack thereof), check the chaos operator logs. Ex:

```bash
kubectl logs -f chaos-operator-ce-6899bbdb9-jz6jv -n litmus  
```

- To view the parameters with which the experiment job is created, status of experiment, success of chaosengine patch operation and cleanup of the experiment pod, check the logs of the chaos-runner pod. Ex:

```bash
kubectl logs sock-chaos-runner -n sock-shop
```

- To view the logs of the chaos experiment itself, use the value `retain` in `.spec.jobCleanupPolicy` of the chaosengine CR

```bash
kubectl logs container-kill-1oo8wv-85lsl -n sock-shop
```

(The detailed troubleshooting faq here: https://docs.litmuschaos.io/docs/faq-troubleshooting/) 

- To re-run the chaosexperiment, cleanup and re-create the chaosengine CR

```bash
kubectl delete chaosengine sock-chaos -n sock-shop
kubectl apply -f litmus/chaosengine.yaml 
```

## List

Lists all the available Litmus Chaos Experiments in this repo under the `./litmus` directory:

```bash
./manage.py list
```

## Shutdown

To shutdown and destroy the GKE cluster when you're finished:

```bash
./manage.py stop --project {GC_PROJECT}
```
