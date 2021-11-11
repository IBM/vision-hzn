## MVI Vision Edge Application Deployment using IEAM

### Setup these ENVs to register the edge device node 
Set these VARs manually or put them in a file to source them.

```
### Prefix for the services naming convention 
export EDGE_OWNER=ibm.edge
export EDGE_DEPLOY=example.vision-hzn

# Used in initialization
export VE_HOSTNAME=`hostname -f`

totmemkb=$(cat /proc/meminfo | grep MemTotal | awk '{ print $2 }')
totmemmb=$(expr $totmemkb / 1000)
ctlrshmem=$(expr $totmemmb / 4)
export VE_CTRLSHMEM=$ctlrshmem
```

#### Register the edge device node as `Complete Vision Edge Application`
- Copy these two files to your node from this repo 

  - node.policy.cme.full.json
  - user.input.cme.full.json

- Run following commmand to deploy full application.
```
hzn register --policy node.policy.cme.full.json -f user.input.cme.full.json
```

- Verity the deployment with `hzn agreement list` and `docker ps` . You should see four containers running similar to 
```
docker ps
CONTAINER ID   IMAGE                                             COMMAND                 CREATED              STATUS              PORTS                                                                  NAMES
b9361fa34575   us.icr.io/ieam-mvi/vision-edge-cme_amd64          "sh -c $CME_MGR_EXE"    About a minute ago   Up About a minute                                                                          896061062736eb4383c06f7bf5f45d00a373059cc955f40427bc46f40b9ecbb4-vision-edge-cme
950be7d40f8f   us.icr.io/ieam-mvi/vision-edge-controller_amd64   "/startController.sh"   About a minute ago   Up About a minute   0.0.0.0:443->443/tcp, 0.0.0.0:5432->5432/tcp, 0.0.0.0:8883->8883/tcp   dev_vision-edge-controller_8.4.0_acf4b431-1ae1-4584-9723-b936159c4a69-vision-edge-controller
a88a17b5a272   us.icr.io/ieam-mvi/vision-edge-dle_amd64          "sh -c $DLE_APP"        About a minute ago   Up About a minute   0.0.0.0:9001->9001/tcp                                                 dev_ibm.edge.example.vision-hzn.vision-edge-dle_8.4.0_9fd54f6a-10fa-4f79-97cb-721e08ba85d5-ibm.edge.example.vision-hzn.vision-edge-dle
b0b28937633a   us.icr.io/ieam-mvi/vision-edge-inception_amd64    "/inception.sh"         About a minute ago   Up About a minute                                                                          dev_ibm.edge.example.vision-hzn.vision-edge-inception_8.4.0_5fb88745-8777-4627-91e8-a1b2d656c62e-ibm.edge.example.vision-hzn.vision-edge-inception
```

- Access the application using URL
The first run of the application creates the username and password and is logged in root protected `/var/log/syslog` file. Run following command before registering the node for the first time to `grep` the credentials.
```
sudo tail -f /var/log/syslog | grep -A 3 'username and password'
```

```
https://<vm.public.ip.address/
``` 

#### Register the edge device node as `Standalone Detector Only` (You may not want to use this option. Test purpose only)
- Copy these two files to your node from this repo 

  - node.policy.dle.detect.json
  - user.input.dle.detect.json

Run following commmand to deploy detector only 
```
hzn register --policy node.policy.dle.detect.json -f user.input.dle.detect.json
```
- Verity the deployment with `hzn agreement list` and `docker ps` . You should see two containers running similar to 
```
docker ps
CONTAINER ID   IMAGE                                            COMMAND            CREATED         STATUS         PORTS                    NAMES
13d8d4999565   us.icr.io/ieam-mvi/vision-edge-dle_amd64         "sh -c $DLE_APP"   2 minutes ago   Up 2 minutes   0.0.0.0:9001->9001/tcp   a38793f52781f38fa900e344257ae378ca2a278a0f33522713698927477a34ad-ibm.edge.example.vision-hzn.vision-edge-dle
907297c666c4   us.icr.io/ieam-mvi/vision-edge-inception_amd64   "/inception.sh"    2 minutes ago   Up 2 minutes                            dev_ibm.edge.example.vision-hzn.vision-edge-inception_8.4.0_ec3703fc-5319-45ad-962c-6a5199c8a3a1-ibm.edge.example.vision-hzn.vision-edge-inception
```

- Access the application using CLI commands

#### DLE client script
Download a copy and keep it ready on the node.
https://github.ibm.com/aivision/vision-edge-dle/tree/master/client/python

#### Sample Test Commands
##### Checks if DLE (detector) is running 
```
python3 dle_client_py/liveready.py 
```

##### Reports info about the model etc
```
python3 dle_client_py/info.py
```
##### Deploys model locally
```
sudo python3 dle_client_py/deploy-local.py -i <path-to-vision-edge-root-directory> -p 9001 -z <absolute-path-to-mode-file> -m <model-id>

e.g.
sudo python3 dle_client_py/deploy-local.py -i /var/local/horizon/vision-edge -p 9001 -z <model.zip> -m <model-id>
```
##### Run a test to detect objects for the given model and inout file
```
python3 dle_client_py/detect.py --model-id <model-id> --image-file <test-jpg-file> 

e.g:
python3 dle_client_py/detect.py --model-id f5c3e1a6-797f-47ee-a139-94ae8f6dc346 --image-file ./Numbers-selected/0019.jpg 
```

