# Tenable Agent - K8s Daemonset

**First Problem:** "I need to start a manual process of installing the tenable.io agent everytie that a node joins my cluster"
**Second Problem:** "I need to implement tenable.io agent in a pod and use this pod to scan the node filesystem"

**Solution:** Create an unprivilleged DaemonSet POD with tenable.io agent and change the filesystem root of the process to the node filesystem

## Description

This repository contains Kubernetes resource files for deploying the Tenable agent as a DaemonSet in a Kubernetes cluster. The Tenable agent runs on all nodes in the cluster and provides visibility into the security posture of your cluster.

## Prerequisites

- [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets) install and use your public key to sign your key.

- [kubectl](https://kubernetes.io/docs/reference/kubectl/) installed and configured to access your cluster.
- [Tenableio link key](https://cloud.tenable.com/tio/app.html#/settings/sensors/agents/agents-list/add)

## Deployment

First, you need to create a **Sealed Secrets** key inside your cluster, if you check the [secrets.yaml](https://github.com/RocketChat/tenable-agent-kubernetes-daemonset/blob/main/secrets.yaml) file you will see that inside this file there's a base64 encoded string and if you run a `base64 -d` the following result will show:

`{"link":{"host": "cloud.tenable.com","port": 443,"key": "YOUR TENABLE KEY GOES HERE","name": "$NODE_NAME", "groups": ["agent-group"]}}`

Change the `YOUR TENABLE KEY GOES HERE` to your tenable.io sync token and convert it again to a base64 format

`echo -n '{"link":{"host": "cloud.tenable.com","port": 443,"key": "YOUR TENABLE KEY GOES HERE","name": "$NODE_NAME", "groups": ["agent-group"]}}' | base64`

Insert in the `secrets.yaml` file and use kubeseal to encrypt it.

```
cat secret.yaml | kubeseal \
    --controller-namespace kube-system \
    --controller-name sealed-secrets \
    --format yaml \
    > sealed-secret.yaml
```

And after that, apply the sealed secrets into your cluster:

`kubectl apply -f sealed-secret.yaml`

Now you can run your daemonset with the following command

`kubectl apply -f manifest/nessus-pod-yaml`

If everything goes well in some minutes you will see your node information in your [tenable.io sensors list](https://cloud.tenable.com/tio/app.html#/settings/sensors/agents/agents-list)


![Image from tenable.io agent list](https://user-images.githubusercontent.com/38704234/210657051-c3be97e5-b7ae-4e31-b2b1-6923421b83c7.png)


If you want to build the image in other registry:

**Important: Remember to change the reference of the image in `nessus-pod.yaml` if you build the dockerfile locally**

```
cd Dockerfile
docker build -t tenable-agent-image . 
```

## Testing

To verify that the Tenable agent is running on all nodes in the cluster, run:

`kubectl get pods -n nessus`

You should see one pod for each node in the cluster, with a status of `Running`.

If you encounter any issues, you can check the logs for the Tenable agent pods using:

`kubectl logs -n nessus <pod-name>`

## Maintenance

To update the Tenable agent `image`, modify the image field in the `nessus-pod.yaml` file and apply the changes using `kubectl apply -f nessus-pod.yaml`.