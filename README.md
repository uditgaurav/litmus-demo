# Litmus Kubernetes Demo Environment

The purpose of this repository is to familiarize oneself with running litmus chaos experiments in a realistic app environment running multiple services on different Kubernetes clusters. 

It makes to spin up a fully deployed [GKE](https://cloud.google.com/kubernetes-engine/) cluster easy with a microservice application or even you can spin up a KinD (Kubernetes-in-Docker) cluster which is a lightweight easy to use and handle for the applications and performing chaos.
[Sock Shop](https://github.com/microservices-demo/microservices-demo), and 
[Litmus Chaos Engine](https://litmuschaos.io/) to create chaos scenarios.

After cloning this repository, installing the requirements listed below, and using the `start` command to create the fully deployed cluster, you will be able to run Litmus Chaos experiments using the `test` command in the cluster. You can find all the experiment configuration under the `/litmus` directory of this repository and the script to deploy and run them in `manage.py`.

It currently works with KinD and GKE so either you can use a KinD cluster by following the below steps or you would need a Google Cloud account to run this on GKE environment and the support for Amazon and Azure is planned in future.

## Requirements

1. Python 3.7 or above
2. Python Dependencies: `pip install -r requirements.txt`
3. Google Cloud Login: https://console.cloud.google.com/
4. GCloud CLI installed locally and logged in: https://cloud.google.com/sdk/docs/quickstarts
5. Kubectl installed locally: https://kubernetes.io/docs/tasks/tools/install-kubectl/
6. Helm installed locally: https://helm.sh/docs/intro/install/
7. Docker installed locally: https://docs.docker.com/engine/install/


## Usage

To see full command-line options use the `-h` flag:

```bash
./manage.py -h
```

This will output the following:

```bash
usage: manage.py [-h] {start,test,list,stop} ...

Spin up Litmus Demo Environment on Kubernetes.

positional arguments:**
  {start,test,list,stop}
    start               Start a Cluster with the demo environment deployed.
    test                Run Litmus ChaosEngine Experiments inside litmus demo
                        environment.
    list                List all available Litmus ChaosEngine Experiments
                        available to run.
    stop                Shutdown the Cluster with the demo environment
                        deployed.
```

## Startup

To start the GKE cluster and deploy all the required components:

**_for kind cluster_**
```bash
./manage.py start --platform kind 
```

**_for GKE cluster_**
```bash
./manage.py start --platform GKE --project {GC_PROJECT} --key {ZE_KEY}
```

**Flag values for start**
<table>
<tr>
<th> Flag </th>
<th> Description </th>
<th> Default </th>
</tr>
<tr>
 <td> <code>--platform</code> or <code>-pt</code>  </td>
 <td> Set the platform to start with demo enviroment. Available platforms are kind and GKE. Support for other platforms will also be added. </td>
 <td>  Default value is <code>kind</code></td>
 </tr>
<tr>
 <td> <code>--name</code> or <code>-n</code> </td>
 <td> Required when <code>--platform</code> is GKE. It sets GKE cluster name </td>
 <td>  Default value is <code>litmus-k8s-demo</code></td>
 </tr>
<tr>
 <td> <code>--zone</code> or <code>-z</code> </td>
 <td> Required when <code>--platform</code> is GKE. It sets GCloud Zone to spin GKE cluster up in </td>
 <td>  Default value is <code>us-central1-a</code></td>
 </tr>
 <tr>
 <td> <code>--project</code> or <code>-p</code> </td>
 <td> Required when <code>--platform</code> is GKE. It sets GCloud Project to spin GKE cluster up in </td>
 <td>  No Default value</td>
 </tr>
 </table>

## Test

To run all the Litmus ChaosEngine experiments:

```bash
./manage.py test
```
You can optionally add the `--wait=` argument to change the wait time between experiments in minutes. By default,
it is 1 min.

To run a specific experiment (found under the ./litmus directory):

```bash
./manage.py test --test=pod-delete
```

**Flag values for test**
<table>
<tr>
<th> Flag </th>
<th> Description </th>
<th> Default </th>
</tr>
<tr>
 <td> <code>--test</code> or <code>-t</code>  </td>
 <td> Name of test to run based on yaml file name under /litmus folder. </td>
 <td>  Default value is <code>*</code> (all)</td>
 </tr>
<tr>
 <td> <code>--wait</code> or <code>-w</code> </td>
 <td> Number of minutes to wait between experiments. </td>
 <td>  Default value is <code>1</code> (in min)</td>
 </tr>
<tr>
 <td> <code>--type</code> or <code>-ty</code> </td>
 <td> Select the type of chaos to be performed, it can have values pod for pod level chaos,node for infra/node level chaos and all to perform all chaos. </td>
 <td>  Default value is <code>all</code> </td>
 </tr>
 <tr>
 <td> <code>--platform</code> or <code>-pt</code> </td>
 <td> Set the platform to perform chaos. Available platforms are kind and GKE. </td>
 <td> Default value is <code>kind</code></td>
 </tr>
  <tr>
 <td> <code>--report</code> or <code>-r</code> </td>
 <td> Set report flag to yes for generating pdf report of the experiment result summary </td>
 <td> Default value is <code>no</code></td>
 </tr>
 </table>

### Notes

- To view application deployment picked, success/failure of reconcile operations (i.e., creation of chaos-runner pod or lack thereof), check the chaos operator logs. Ex:

```bash
kubectl logs -f chaos-operator-ce-6899bbdb9-jz6jv -n litmus  
```

- To view the parameters with which the experiment job is created, the status of experiment, the success of chaosengine patch operation, and cleanup of the experiment pod, check the logs of the chaos-runner pod. Ex:

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

## Generate PDF of the experiment result summary

We can also generate the pdf report of the experiment result summary using <code>--report</code> flag as follow:

```bash
./manage.py test --report=yes
``` 
It will generate a pdf report of name `chaos-report.pdf` in the current location containing chaos result summary.

## List

Lists all the available Litmus Chaos Experiments in this repo under the `./litmus` directory for a particular platform:

```bash
./manage.py list --platform <platform-name>
```


## Shutdown

To shut down and destroy the cluster when you're finished:

**_for kind cluster_**
``` bash
./manage.py --platform kind stop
```

**_for GKE cluster_**
```bash
./manage.py --platform GKE stop --project {GC_PROJECT}
```
