# Installing downstream (pre-GA) 3rd level Operators with OperatorHub/OLM
## Get access to rh-osbs-operators registry
Put your name and quay.io info here:  https://docs.google.com/spreadsheets/d/1OyUtbu9aiAi3rfkappz5gcq5FjUbMQtJG4jZCNqVT20/edit#gid=0
## Use @anli's tools to synch operator images to your cluster
- Run from a shell with KUBECONFIG set that has **oc** access to your cluster
- Clone the v3-testfiles repo
```sh
$ git clone http://git.app.eng.bos.redhat.com/git/openshift-misc.git/
$ cd jenkins/v4-image-test/app_registry_tools
```
- **make sure Python 3 is active either as the system python or via virtualenv**
- edit `get_medadata_from_app_registry.sh` and set REPOSITORYS for the operators you want.  
Examples:

```
REPOSITORYS="nfd elasticsearch-operator cluster-logging openshifttemplateservicebroker openshiftansibleservicebroker sriov-network-operator"
REPOSITORYS="nfd elasticsearch-operator cluster-logging"
```
- Run **./get_medadata_from_app_registry.sh rh-osbs-operators 4.2**   (if you see an AttributeError complaining about no FullLoader,  you ignored the step on Python 3 above)  
*NOTE: This script is actively being developed and may require some tweaking to work.*  
- That will download the metadata for the operator.  Next, mirror the images to your cluster - may take a while depending on the number of images
- **./sync_images_to_internal_registry.sh rh-osbs-operators 4.2**

## Configure your operator sources on your cluster
First you need to get a login token for quay.io after you have been added to the rh-osbs-operators group

- Get your quay user token using command (only have to do this once, can reuse the token when creating the secret on subsequent clusters):

```sh
QUAY_USERNAME=
QUAY_PASSWORD=''
curl -sH "Content-Type: application/json" -XPOST https://quay.io/cnr/api/v1/users/login -d '{ "user": { "username": "'"${QUAY_USERNAME}"'","password": "'"${QUAY_PASSWORD}"'"}}' | jq -r '.token' | tee quay.token
```

- Create a secret with that token data in the openshift-marketplace namespace example yaml to use:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: marketplacesecret
  namespace: openshift-marketplace
type: Opaque
stringData:
    token: <put_quay_token_here>
```
- Disable the default operator sources.  **oc edit OperatorHub** and make the spec  look like:
```yaml
apiVersion: config.openshift.io/v1
kind: OperatorHub
metadata:
  name: cluster
spec:
  disableAllDefaultSources: true
```
- Finally create a new operatorsource for rh-osbs-operator.   Create a yaml file with the following and run **oc create -f** against it.
```yaml
apiVersion: operators.coreos.com/v1
kind: OperatorSource
metadata:
  name: rh-osbs-applications
  namespace: openshift-marketplace
spec:
  type: appregistry
  endpoint: https://quay.io/cnr
  registryNamespace: rh-osbs-operators
  authorizationToken:
    secretName: marketplacesecret
```

- verify it worked:
```sh
oc get operatorsource -n openshift-marketplace
NAME                   TYPE          ENDPOINT              REGISTRY            DISPLAYNAME   PUBLISHER   STATUS      MESSAGE                                       AGE
rh-osbs-applications   appregistry   https://quay.io/cnr   rh-osbs-operators                             Succeeded   The object has been successfully reconciled   27m
```
## Install your operator
Go to OperatorHub and install.  Phew.