# GPU Enablement on Openshift

Open Data Hub provides us support for accessing NVIDIA GPUs from Jupyter notebooks which helps users in training their ML models using the speed & power of GPUs.

Now, Enabling of GPUs in Openshift 4.x can be done using the following two operators:

* [Node Feature Discovery (NFD) Operator](https://github.com/openshift/cluster-nfd-operator): The Node Feature Discovery operator is responsible primarily for managing the detection of hardware features and configuration of various nodes present in an Openshift cluster. It discovers and labels the hardware features (such as GPU(s)) available on each of the node.

* [Nvidia GPU Operator](https://github.com/NVIDIA/gpu-operator): This is the operator which is responsible for actually installing the GPU drivers on the corresponding nodes which contains GPUs in the cluster. Configuring and managing the nodes with GPU resources requires the configuration of multiple software components such as drivers, container runtimes or other libraries which is all done by the Nvidia GPU Operator in Openshift.


## Requirements:

1. Before proceeding with the installation of the Nvidia GPU Operator, we need to make sure that we have appropriate RHEL entitlements created & deployed in our cluster. This is required to download various packages used to build the driver container of the GPU operator. We can leverage the features of entitled builds on any non-RHEL host as long as some of the prerequisites are fulfilled in the container. Now, generally when using a UBI based container (which is used by Nvidia GPU Operator), the only missing part is the subscription certificate. So, we need to first have them downloaded.

2. Node Feature Discovery Operator should be installed and its instance needs to be created prior to the deployment of the Nvidia GPU Operator so that NFD is able to label all the nodes according to their hardware configurations and these labels are then used by the Nvidia GPU Operator to identify the nodes having GPUs resources and configure all the required drivers, packages and libraries on them.


## Instructions:

#### Enabling Cluster-Wide Entitlement Builds on Openshift:

1. If you already have cluster-wide RHEL entitlements setup across your cluster, then you can skip to step 6. If not then continue with the next step.

2. Login to your Redhat account using which the openshift cluster has been created and then click on the **Customer Portal** tab on the top. Then click on the **Subscriptions** tab from the top navbar and move to the **Systems** section under this.

3. From there, you can create & download the appropriate subscription certificates which are going to be used then for creating cluster-wide entitlements in the next step.

4. Now, run the following commands to create a MachineConfig in the openshift cluster for enabling cluster-wide entitlement builds.

```
$ cp <path/to/pem/file>/<certificate-file-name>.pem nvidia.pem

$ curl -O  https://raw.githubusercontent.com/openshift-psap/blog-artifacts/master/how-to-use-entitled-builds-with-ubi/0003-cluster-wide-machineconfigs.yaml.template

$ sed "s/BASE64_ENCODED_PEM_FILE/$(base64 -w0 nvidia.pem)/g" 0003-cluster-wide-machineconfigs.yaml.template > 0003-cluster-wide-machineconfigs.yaml

$ oc create -f 0003-cluster-wide-machineconfigs.yaml
```

5. You can validate the creation of entitlements by running:

```
oc get machineconfig | grep entitlement
```

## Installing Nvidia GPU Operator and Node Feature Discovery (NFD) Operator:

6. The NFD Operator gets installed automatically with the Nvidia GPU Operator. So, we need not install it separately using the Operator Hub.

7. Create a new namespace named "gpu-operator-resources". This is the namespace under which the GPU Operator resources are going to be installed.

8. Move to the Operator Hub and search for the Nvidia GPU Operator. Click on **Install**. Next, we are going to install the Operators inside a specific namespace, viz. "gpu-operator-resources".

<img src="./images/Image - 1.png" width="800px" alt="Screen Shot 1 here">
<br>

9. Now, we can see both the Nvidia GPU Operator as well as NFD operators would be installed.

<img src="./images/Image - 2.png" width="800px" alt="Screen Shot 2 here">
<br>

10. Now, create an instance of NFD Operator and it will automatically label all the hardware nodes present in our cluster.

<img src="./images/Image - 3.png" width="800px" alt="Screen Shot 3 here">
<br>

11. Next, create an instance of the Nvidia GPU Operator and it would deploy all the required software components including the NVIDIA drivers (to enable CUDA), Kubernetes device plugin for GPUs, the NVIDIA Container Runtime and DCGM based monitoring.

<img src="./images/Image - 4.png" width="800px" alt="Screen Shot 4 here">
<br>

12. We can see now all of the appropriate Nvidia GPU Operator pods would be deployed.

<img src="./images/Image - 5.png" width="800px" alt="Screen Shot 5 here">
