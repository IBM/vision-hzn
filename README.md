### Maximo Visual Inspection (MVI) Vision-Edge deployment using IBM Edge Application Manager(IEAM)

MVI Vision Edge is an edge deployable application and is used to do machine inferencing at the edge. This works with cloud deployed training server.

In this `IEAM based deployment` the MVI Vision Edge application deployment is fully automated using edge node policy.

Typical deployment of MVI Vision Edge requires setting up the edge node using a docker script that creates several directories to store configuration and database on the edge node. Another `startedge` script installs three additional docker containers using combination of docker run, docker network, docker exec and several other CLI commands. IEAM based deployment simplifies the approach.

The application has four docker containers:
- Inception 
- DLE
- Controller
- CME

Additionally, IEAM based deployment can be done in two modes:

1. Vision Edge Detector only - This deploys the main DLE detector container in addition to initial setup and provides full capability to run ML tasks (inferencing, classification, detection etc) based on separately deployed ML model. 

2. Complete Vision Edge Application - In addition to above containers, controller and CME are also installed to provide UI and integration with cloud deployed training server for model download, configuration, setting up inspection stations etc. 

### MVI Vision-Edge Deployment 

#### Pre-requisite - `MUST DO`
**Note:** Instruction to setup the edge device node is longer than the actual running of the application. You might have already done these steps. Review the steps and complete them as necessary.

1a. This installation is currently tested on 

- Ubuntu VM + P100 - x86/amd64 available in IBM Cloud. Setup an appropriate VM in IBM cloud under your account.
- NVidia Jetson Xavier NX - nvidia-jetpack 4.5.1-b17

2. Login as `root`. Setup a non-root user account on the VM. Application will be run using `non-root` user. 
```
useradd -s /bin/bash -d /home/<username> -m -G sudo <username>
usermod -g users <username>
passwd <username>
```
3. Logout and log back in as `non-root` user. 

4. Install IEAM Edge Agent using the provided credentials as `non-root` user. Start by setting up the ENV variables. 
```
export HZN_ORG_ID=<HZN_ORG_ID>
export HZN_EXCHANGE_USER_AUTH=iamapikey:<API_KEY_PROVIDED>
export HZN_EXCHANGE_URL=https://cp-console.ieam42-edge-8e873dd4c685acf6fd2f13f4cdfb05bb-0000.us-south.containers.appdomain.cloud/edge-exchange/v1
export HZN_FSS_CSSURL=https://cp-console.ieam42-edge-8e873dd4c685acf6fd2f13f4cdfb05bb-0000.us-south.containers.appdomain.cloud/edge-css
curl -u "$HZN_ORG_ID/$HZN_EXCHANGE_USER_AUTH" -k -o agent-install.sh $HZN_FSS_CSSURL/api/v1/objects/IBM/agent_files/agent-install.sh/data
chmod +x agent-install.sh
sudo -s -E ./agent-install.sh -i 'css:'
```
**Reference:** https://www.ibm.com/support/knowledgecenter/SSFKVV_4.2/installing/automated_install.html

5. Add above non-root user in docker group
```
sudo usermod -aG docker <username>
```
6. Logout and login back in for docker group to become effective.

7. This step applies to Ubuntu VM + P100 and NOT Jetson NVidia Xavier NX. 

   Install/update nvidia driver on the VM. Verify that the following command works and shows similar output. If not then may follow steps outlined below to fix.
```
nvidia-docker run --rm nvidia/cuda:10.2-base-ubuntu18.04 nvidia-smi
```
Output
```
Thu Sep 16 23:55:34 2021       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 460.91.03    Driver Version: 460.91.03    CUDA Version: 11.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Tesla P100-PCIE...  Off  | 00000000:00:07.0 Off |                    0 |
| N/A   35C    P0    27W / 250W |      0MiB / 16280MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```
**Steps to update nvidia drivers and docker**
```
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update
sudo apt-get install nvidia-docker2
sudo systemctl restart docker.service
```
Run following command to verify `nvidia-smi` and you should see output as above. 
```
nvidia-docker run --rm nvidia/cuda:10.2-base-ubuntu18.04 nvidia-smi
```
*If not successful then reboot the VM and Run above command again before purging and starting the install again.

You may also try searching for nvidia-driver versions such as 460, 470 and install one of them.
```
sudo apt search nvidia-driver
```

*If not successful you may have to purge your nividia install and redo the steps as follows*
```
sudo apt-get purge nvidia*
sudo apt install nvidia-driver-460-server
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update
sudo apt-get install nvidia-docker2
sudo systemctl restart docker.service
nvidia-docker run --rm nvidia/cuda:10.2-base-ubuntu18.04 nvidia-smi
```
8. Update docker runtime to use nvidia-docker by modifying the `/etc/docker/daemon.json` with the following content. You are adding this line `"default-runtime" : "nvidia"`
```
{
    "default-runtime" : "nvidia",
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}
```
9. Restart docker
```
sudo systemctl restart docker.service
```

#### Reference:
MVI Vision-Edge Document 
https://www.ibm.com/docs/en/maximo-vi/8.4.0

https://www.ibm.com/docs/en/maximo-vi/8.4.0?topic=planning-installing-docker-nvidia-docker2 

### MVI - Vision Edge deployment
`DO NOT move forward` until the edge node infrastruture is properly setup as above - `nvidia-smi` and `/etc/docker/daemon.json`

#### To Register Node
**NOTE:** Services and policies must be already published into IEAM Mgmt Hub under your organization. Click here for instructions to `register` the edge device node with MVI - Vision Edge Application
https://github.com/IBM/vision-hzn/tree/main/register

#### To Publish Service and Policy
Click here for instructions to `publish` the `services and policies`
https://github.com/IBM/vision-hzn/tree/main/publish

