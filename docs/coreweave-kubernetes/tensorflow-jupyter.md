# README

## Tensorflow with Jupyter

![Screenshot](../.gitbook/assets/screenshot%20%281%29.png)

### Introduction

This example leverages a [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) to always maintain one instances of [Tensorflow](https://www.tensorflow.org) with Jupyter. Tensorflow is a highly popular deep learning framework that is greatly accelerated by GPUs. Each instance, in Kubernetes terminology called a [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) is allocated 2 GPUs.

The Kubenetes Control-Plane in the CoreWeave Cloud will ensure that there are one instance \([Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/)\) of Tensorflow Jupyter running at all times. The Control-Plane will reserve GPU, CPU and RAM on CoreWeaves compute nodes. Pods in the same deployment can be scheduled on the same or multiple physical nodes, depending on resource availability. If co-location of Pods is required for some reason, ie. shared ephemeral or block storage, this can be controlled with [affinity rules](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity).

The example Deployment does showcase some node affinity rules. These are purely for demonstration purposes, and the entire affinity section can be removed without breaking the example.

### Service

A [Service](https://kubernetes.io/docs/concepts/services-networking/service/) is included to show how to publish a Deployment to the public Internet. The Service publishes the web interface of Jupyter to the Internet on port `8888`.

## Persistent Storage

A [Persistent Volume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) is allocated to persist user uploaded Notebooks. The allocation is done via a [Persistent Volume Claim](https://github.com/atlantic-crypto/kubernetes-cloud-examples/blob/master/cuda-ssh/sshd-pvc.yaml) requesting the storage size and backing storage type \(SSD, HDD\). This volule claim is then mounted to the `/tf/notebooks` directory in the Pod definition. Utilizing a persistent volume ensures that files persist even if the node currently running the Pod fails.

### Getting Started

After installing `kubectl` and adding your CoreWeave Cloud access credentials, the following steps will deploy the Ethminer Deployment and service.

1. Apply the resources. This can be used to both create and update existing manifests

   ```text
    $ kubectl apply -f tensorflow-deployment.yaml
    deployment.apps/tensorflow-jupyter configured
    $ kubectl apply -f tensorflow-service.yaml
    service/tensorflow-jupyter configured
   `
   ```

2. List pods to see the Deployment working to instantiate all our requested instances

   ```text
    $ kubectl get pods
    NAME                                 READY   STATUS              RESTARTS   AGE
    tensorflow-jupyter-6794bcb465-4czqb  0/1     ContainerCreating   0          2s
   ```

3. After a little while, all pods should transition to the `Running` state

   ```text
   $ kubectl get pods
   NAME                                  READY   STATUS    RESTARTS   AGE
   tensorflow-jupyter-6794bcb465-4czqb   1/1     Running   0          6s
   ```

4. The Deployment will also show that all desired Pods are up and running

   ```text
    $ kubectl get deployment
    NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
    tensorflow-jupyter   1/1     1            1           73m
   ```

5. Describing a Pod will help troubleshoot a Pod that does not want to start and gives other relevant information about the Pod

   ```text
    $ kubectl describe pod tensorflow-jupyter-6794bcb465-4czqb
    ....
    Events:
      Type    Reason                  Age   From                     Message
      ----    ------                  ----  ----                     -------
      Normal  Scheduled               54s   default-scheduler        Successfully assigned tenant-test/tensorflow-jupyter-6794bcb465-4czqb to g04c225
      Normal  SuccessfulAttachVolume  54s   attachdetach-controller  AttachVolume.Attach succeeded for volume "pvc-66fa0887-a4c4-4245-a4e2-02200f640fea"
      Normal  Pulling                 50s   kubelet, g04c225         Pulling image "tensorflow/tensorflow:1.15.0-py3-jupyter"
      Normal  Pulled                  24s   kubelet, g04c225         Successfully pulled image "tensorflow/tensorflow:1.15.0-py3-jupyter"
      Normal  Created                 19s   kubelet, g04c225         Created container miner
      Normal  Started                 18s   kubelet, g04c225         Started container miner
   ```

6. The logs can be viewed to retrieve the Jupyter login token

   \`\`\`shell $ kubectl logs tensorflow-jupyter-6794bcb465-4czqb

   **\_** /**\_\_** **\_\_/** /_**\_\_**_  **** /  __ \_  **\_ \_**/  **\_ \_**/ _/_  **/\_**  \_ \| /\| / /  _/ / **/ / / /\(** \)/ /_/ / /  _\_\_/_  / / /_/ /_ \|/ \|/ / /_/ \_\__//_/ /_//**/ \**//_/ /_/ /_/ \_**/\_**/\|\__/

```text
WARNING: You are running this container as root, which can cause new files in
mounted volumes to be created as the root user on your host machine.

To avoid this, run the container by specifying your user's userid:

$ docker run -u $(id -u):$(id -g) args...

[I 14:09:12.985 NotebookApp] Writing notebook server cookie secret to /root/.local/share/jupyter/runtime/notebook_cookie_secret
[I 14:09:13.153 NotebookApp] Serving notebooks from local directory: /tf
[I 14:09:13.153 NotebookApp] The Jupyter Notebook is running at:
[I 14:09:13.153 NotebookApp] http://tensorflow-jupyter-6794bcb465-4czqb:8888/?token=a71eb39261e6ef01bdec8867c2c051b0b3aaf31545bfbb84
[I 14:09:13.153 NotebookApp]  or http://127.0.0.1:8888/?token=a71eb39261e6ef01bdec8867c2c051b0b3aaf31545bfbb84
```
```

To get the public IP assigned to the service, simply list all services

```text
$ kubetl get service                                                                                                                                                                                                                               git:(master↓3|…
NAME                 TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)          AGE
tensorflow-jupyter   LoadBalancer   10.134.100.173   64.79.105.199   8888:30947/TCP   30s
```

\`\`\`

You can now now access Jupyter on `http://EXTERNAL-IP:8888` using the login token from the log.

The package includes Tensorflows example notebooks that will leverage the GPUs available to the Pod.
