# Extracting the openshift-tests binary from a nightly build
## Get a token for registry.svc.ci.openshift.org if you do not have one
1. Go to https://api.ci.openshift.org/console/catalog, click your name in upper right and then Copy Login Command
1. Run the copied login command.  Example:  **oc login https://api.ci.openshift.org --token=my_token**
1. Login to the registry: **podman login -u mifiedle -p $(oc whoami -t) registry.svc.ci.openshift.org**  replace mifiedle with your id

## Pull the tests image and extract the binary
* Find the desired build on the [release page](https://openshift-release.svc.ci.openshift.org/). Example: **4.1.0-0.nightly-2019-05-16-090009**
* Pull the image tagged "tests":  **podman pull registry.svc.ci.openshift.org/ocp/4.1-art-latest-2019-05-16-090009:tests**  Note that the image name is different from the build name.   The timestamp is the part to copy over to the pull.
* Start a container with the image:  **podman run registry.svc.ci.openshift.org/ocp/4.1-art-latest-2019-05-16-090009:tests**   It will exit without doing anything.
* Get the container id of the Exited container:  **podman ps -a**   
```bash
# podman ps -a
CONTAINER ID  IMAGE                                                                     COMMAND    CREATED        STATUS                    PORTS  NAMES
dd0d7ecaa512  registry.svc.ci.openshift.org/ocp/4.1-art-latest-2019-05-16-090009:tests  /bin/bash  4 minutes ago  Exited (0) 4 minutes ago         condescending_allen
```
* Mount the container's filesystem 
```bash
# podman mount dd0d7ecaa512
/var/lib/containers/storage/overlay/ab267f105002aada3cc02968093b96da1078fe1105563ec3683318af85c1700c/merged
```
* copy the openshift-tests binary
```bash
cp /var/lib/containers/storage/overlay/ab267f105002aada3cc02968093b96da1078fe1105563ec3683318af85c1700c/merged/usr/bin/openshift-tests /my/dir/openshift-tests
```
* **podman unmount container_id**
* **podman rm container_id**