# ovirt-openshift-extensions

[![Build Status](http://jenkins.ovirt.org/buildStatus/icon?job=oVirt_ovirt-openshift-extensions_standard-on-ghpush)](http://jenkins.ovirt.org/job/oVirt_ovirt-openshift-extensions_standard-on-ghpush/)
[![Go Report Card](https://goreportcard.com/badge/github.com/ovirt/ovirt-openshift-extensions)](https://goreportcard.com/report/github.com/ovirt/ovirt-openshift-extensions)

| container image | status    | 
| :---            | :---      |
|ovirt-flexvolume-driver |[![ovirt-flexvolume-driver](https://quay.io/repository/rgolangh/ovirt-flexvolume-driver/status)](https://quay.io/repository/rgolangh/ovirt-flexvolume-driver/status) |
|ovirt-volume-provisioner|[![ovirt-volume-provisioner](https://quay.io/repository/rgolangh/ovirt-volume-provisioner/status)](https://quay.io/repository/rgolangh/ovirt-volume-provisioner/status) |
|ovirt-cloud-provider    |[![ovirt-cloud-provider](https://quay.io/repository/rgolangh/ovirt-cloud-provider/status)](https://quay.io/repository/rgolangh/ovirt-cloud-provider/status) |
|ovirt-flexvolume-driver-apb | [![Docker Repository on Quay](https://quay.io/repository/rgolangh/ovirt-flexvolume-driver-apb/status "Docker Repository on Quay")](https://quay.io/repository/rgolangh/ovirt-flexvolume-driver-apb) |
|ovirt-openshift-installer    | [![Docker Repository on Quay](https://quay.io/repository/rgolangh/ovirt-openshift-installer/status "Docker Repository on Quay")](https://quay.io/repository/rgolangh/ovirt-openshift-installer)

## Purpose
Make oVirt the a prefered platform for openshift installation.
The main components this project will ship are:
  - storage integration through plugin - currently flex, CSI in the future
  - cloud provider
  - easy deployement of all those components

### ovirt-volume-provisioner
A kubernetes controller that creates/deletes persistent volumes, and allocates disks \
in ovirt as a result. This is the first part for providing volumes from oVirt.

### ovirt-flexvolume-driver
A kubernetes node plugin that attaches/detaches a volume to a container. \
It attaches the oVirt disk to the kube node (which is an oVirt VM). It identifies the disk device on the os, \
prepares a filesystem, then mounts it so it is ready as a volume mount for a container.

### ovirt-cloud-provider
An out-of-tree implementation of a cloudprovider. \
A controller that manages the admission of new nodes for openshift, from oVirt VMs. \

### Versions
| version   |ovirt version |openshift version|
|-----------|--------------|-----------------|
|\<= v0.3.1 | \>= 4.2      | 3.9,  3.10      |
|\>= v0.3.2 | \>= 4.2      | 3.10, 3.11      |

### Deployment

#### Deploy via service-catalog
 
Pre-requisite:
- Openshift 3.10.0 or higher
- Running service catalog

From the repo:
- push the apb image to your cluster repo
   ```console
   $ make apb_build apb_push
   ```
- go to the service catalog UI and deploy the ovirt-flexvolume-driver-apb. \
 Here is a demo doing that: \
<a href="http://www.youtube.com/watch?feature=player_embedded&v=frcehKUk_g4" target="_blank"><img src="http://img.youtube.com/vi/frcehKUk_g4/0.jpg" alt="IMAGE ALT TEXT HERE" width="240" height="180" border="10" /></a>

#### Deploy via cli

- make sure `oc` command is configured and has access to your cluster, e.g. run `oc status`

- use a cluster admin user to deploy, or grant permission to one
   ```console
   $ oc login -u system:admin
   $ oc adm policy add-cluster-role-to-user cluster-admin developer
   ```
- Run on a master (replace the input with yours):
   ```console
    docker run \
    -it \
    --rm \
    --net=host \
    -v $HOME/.kube:/opt/apb/.kube:z \
    -u $UID quay.io/rgolangh/ovirt-flexvolume-driver-apb \
    provision -e \
    '{
      "admin_user":"developer",
      "admin_password":"YOURPASS",  
      "cluster":"openshift",
      "namespace":"default",
      "engine_username":"admin@internal",
      "engine_password":"YOURPASS",
      "engine_url":"https://ENGINE-FQDN/ovirt-engine/api",
      }'
   ```

  If it's the first time deploying the image then it should take few moments to download it. To customize the images to be deployed, provide these environment variables to the container invocation:
   * FLEX_REGISTRY
   * FLEX_REPOSITORY
   * FLEX_VERSION
   * PROVISIONER_REGISTRY
   * PROVISIONER_REPOSITORY
   * PROVISIONER_VERSION

  Example: `docker run -e FLEX_REGISTRY=registry.example.com ...`


Upon completion you have these components running:

   ```console
   $ oc get ds -n default ovirt-flexvolume-driver 
   name                      desired   current   ready     up-to-date   available   node selector   age
   ovirt-flexvolume-driver   1         1         1         1            1           <none>          15m

   $ oc get deployment -n default ovirt-volume-provisioner 
   NAME                       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
   ovirt-volume-provisioner   1         1         1            1           17m
   ```

### Documentation
For topics not covered in this Readme, see the project [documentation](https://github.com/oVirt/ovirt-openshift-extensions/tree/master/docs).

### Contributing
**Feedback is most welcome**, if you have an idea, proposal, fix, or want to chat \
  you'll find the details here: 
- Patches are usual pull-request, see the pull request template.
- Upstream bugs: https://github.com/oVirt/ovirt-openshift-extensions/issues
- Trello Board: ovirt-openshift-extensions (may change to Jira)
- Bugzilla tracker bug: https://bugzilla.redhat.com/show_bug.cgi?id=1581638 (see the bugzilla tracker for the component under rhev)
- IRC: upstream: #ovirt @ oftc.net
- Mailing List: devel@ovirt.org

Blog post in ovirt: https://www.ovirt.org/blog/2018/02/your-container-volumes-served-by-ovirt/

Youtube Demo: https://youtu.be/_E9pUVrI0hs


### References
- [flexvolume plugin page on openshift](https://docs.openshift.org/latest/install_config/persistent_storage/persistent_storage_flex_volume.html)
- [flexvolume spec on kubernetes page](https://github.com/kubernetes/community/blob/master/contributors/devel/flexvolume.md)

[flex-conf]: deployment/ovirt-flexdriver/ovirt-flexdriver.conf.j2
[flex-playbook]: deployment/ovirt-flexdriver/deploy.yaml
[prov-playbook]: deployment/ovirt-provisioner/deploy.yaml
