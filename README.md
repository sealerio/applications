# sealer Applications

sealer applications contains plenty of useful distributed application samples.
All these samples follow the Cluster Image format. Users could just take them
away, build ClusterImage all by themselves and finally run these distributed
applications on demand.

sealer applications repo shows demos of widely adopted distributed applications,
but it is not possible for sealer community to provide every distribution
application and every version's ClusterImage. Thus, ClusterImage sufficiency is
not the initial purpose of this repo, but the essential guidance of building
ClusterImage. In addition, sealer community provides limited support of all
distributed applications in this repository.

## Search ClusterImage

sealer tool provides convenient way to start a distrbuted application in
ClusterImage. While it still has challenge for user to get satisfied and detailed
ClusterImage, such as MySQL, ElasticSearch and so on. Here is the procedure that
user could follow when searching some spefific ClusterImage:

* Step 1: use `sealer search <APP_NAME>` command to search application <APP_NAME> from remote registry;
* Step 2: if step 1 fails, come to this `applications` repo to search whether <APP_NAME> application exists here;
* Step 3: if step 2 fails either, please file an issue in this repository, describe detailed demand and the community may provide help.

### Apply a cluster

You can modify the image name and save it as "clusterfile.yaml", then execute cmd `sealer apply -f clusterfile.yaml`.

```yaml
apiVersion: sealer.cloud/v2
kind: Cluster
metadata:
  creationTimestamp: null
  name: my-cluster
spec:
  hosts:
    - ips: [ 192.168.0.2 ]
      roles: [ master ] # add role field to specify the node role
      env: # rewrite some nodes has different env config
        - etcd-dir=/data/etcd
      ssh: # rewrite ssh config if some node has different passwd...
        user: xxx
        passwd: xxx
        port: "2222"
    - ips: [ 192.168.0.3 ]
      roles: [ node,db ]
  image: kubernetes:v1.19.8
  ssh:
    encrypted: true
    passwd: xxxxx
    pk: /root/.ssh/id_rsa
    port: "22"
    user: root
```

If you want to apply a ClusterImage which needs persistence storage. We provide openebs as cloud storage backend. OpenEBS provides block volume support through the iSCSI protocol. Therefore, the iSCSI client (initiator) presence on all Kubernetes nodes is required. Choose the platform below to find the steps to verify if the iSCSI client is installed and running or to find the steps to install the iSCSI client.For openebs, different storage engine need to config different prerequisite. more to see [openebs website](https://github.com/openebs/openebs). We provide plugin mechanism, you only need to append below example to "clusterfile.yaml" and apply them together.

For example, if we use jiva engine as storage backend :

```yaml
apiVersion: sealer.aliyun.com/v1alpha1
kind: Plugin
metadata:
  name: SHELL
spec:
  action: PostInstall
  on: role=node
  type: SHELL
  data: |
    if type yum >/dev/null 2>&1;then
    yum -y install iscsi-initiator-utils
    systemctl enable iscsid
    systemctl start iscsid
    elif type apt-get >/dev/null 2>&1;then
    apt-get update
    apt-get -y install open-iscsi
    systemctl enable iscsid
    systemctl start iscsid
    fi
---
```

### Rebuild a ClusterImage

Use it as base image to build another useful image. See each manifest yaml file under application manifest directory for
details , and modify it according to your needs.

For example, use manifest to build a mysql ClusterImage:

Kubefile:

```shell
FROM kubernetes:v1.19.8
COPY mysql.yaml manifests
CMD kubectl apply -f manifests/mysql.yaml
```

Then run below command to rebuild it

```shell
sealer build -t {Your Image Name} -f Kubefile
```