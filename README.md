# Tenable Agent - K8s Daemonset

**First Problem:** "I need to start a manual process of installing the tenable.io agent everytime that a node joins my cluster"  

**Second Problem:** "I need to implement tenable.io agent in a pod and use this pod to scan the node filesystem"

**Solution:** Create an unprivilleged DaemonSet POD with tenable.io agent and change the filesystem root of the process to the node filesystem

## Description

This repository contains Kubernetes resource files for deploying the Tenable agent as a DaemonSet in a Kubernetes cluster. The Tenable agent runs on all nodes in the cluster and provides visibility into the security posture of your cluster.

## Prerequisites

- [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets) must be implemented

- [kubectl](https://kubernetes.io/docs/reference/kubectl/) must be installed and configured to access your cluster
- A [Tenableio link key](https://cloud.tenable.com/tio/app.html#/settings/sensors/agents/agents-list/add) is required to link the agent with your Tenable.io Manager

## Deployment


1. Create a Sealed Secrets key inside your cluster. Replace `YOUR TENABLE KEY GOES HERE` with your tenable.io sync token and convert it to a base64 format:

First, you need to create a **Sealed Secrets** key inside your cluster, if you check the [secrets.yaml](https://github.com/RocketChat/tenable-agent-kubernetes-daemonset/blob/main/secrets.yaml) file you will see that inside this file there's a base64 encoded string and if you run a `base64 -d` the following result will show:

`echo -n '{"link":{"host": "cloud.tenable.com","port": 443,"key": "YOUR TENABLE KEY GOES HERE","name": "$NODE_NAME", "groups": ["agent-group"]}}' | base64`

2. Insert the base64 encoded string in the [secrets.yaml](https://github.com/RocketChat/tenable-agent-kubernetes-daemonset/blob/main/secrets.yaml) file and use kubeseal to encrypt it:

```
cat secret.yaml | kubeseal \
    --controller-namespace kube-system \
    --controller-name sealed-secrets \
    --format yaml \
    > sealed-secret.yaml
```

3. Apply the sealed secrets to your cluster:

`kubectl apply -f sealed-secret.yaml`

4. Deploy the DaemonSet with the following command:

`kubectl apply -f manifest/nessus-pod.yaml`

After a few minutes, you should see your node information in your [tenable.io sensors list](https://cloud.tenable.com/tio/app.html#/settings/sensors/agents/agents-list)


If you want to build the image in another registry, you can use the following command:

```
cd Dockerfile
docker build -t tenable-agent-image . 
```

**Important: Remember to change the reference of the image in `nessus-pod.yaml` if you build the dockerfile locally**

## Testing

To verify that the Tenable agent is running on all nodes in the cluster, run:

`kubectl get pods -n nessus`

You should see one pod for each node in the cluster, with a status of `Running`.

If you encounter any issues, you can check the logs for the Tenable agent pods using:

`kubectl logs -n nessus <pod-name>`

## Maintenance

To update the Tenable agent `image`, modify the image field in the `nessus-pod.yaml` file and apply the changes using `kubectl apply -f nessus-pod.yaml`.