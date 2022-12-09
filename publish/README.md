## Publish MVI Vision Edge Application Service and Policy 

`DO NOT PUBLISH` container images into external docker repo. Use IBM Container Registry only with authentication. `COMPANY CONFIDENTIAL`.

This uses containers from MVI Vision Edge 8.4.0 release. 

1. Setup code signing key (This is to be done once on the dev node)
```
 hzn key create IBM <your-full-email-id@company.com>
```
2. Clone or copy this code locally on your dev machine, 

3. Setup these ENVs to publish the service and policy. Put these ENV vars in a file to source them conveniently. These ENVs include the ones required for register and additional ones.  
```
### Prefix for the services naming convention 
export EDGE_OWNER=<string-to-group-your-services> # e.g: sg.edge
export EDGE_DEPLOY=<string-to-group-your-services> # e.g: example.vision-hzn

# Used in initialization
export VE_HOSTNAME=`hostname -f`

totmemkb=$(cat /proc/meminfo | grep MemTotal | awk '{ print $2 }')
totmemmb=$(expr $totmemkb / 1000)
ctlrshmem=$(expr $totmemmb / 4)
export VE_CTRLSHMEM=$ctlrshmem

## Additional for publishing
# Always this path
export CONTAINER_APP_ROOT=/opt/ibm/vision-edge

# Absolute path to a directory where bind volumes would go. Your choice.
export APP_BIND_HORIZON_DIR=/var/local/horizon

# Absolute path to a directory where vision edge specific files go for IEAM based deployment - config, models, db etc.
export VE_HOST_SERVICE_ROOT=$APP_BIND_HORIZON_DIR/vision-edge

# Authenticated IBM CR access ###
export CR_IBM_HOST=us.icr.io
export CR_IBM_USERNAME=iamapikey
export CR_IBM_HOST_NAMESPACE=ieam-mvi
export CR_IBM_API_KEY_RO_PULL=<api-key-read-only>
```
4. Run `make` to build and publish service and policy.
```
make
```

5. See `register` section to register the node.

