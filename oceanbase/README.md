# Overview
The ob-operator allows OceanBase to run seamlessly on public cloud or privately deployed Kubernetes clusters in the form
of containers.See [ob-operator installation](https://github.com/oceanbase/ob-operator/blob/master/README.md)

## How to use it

1, Three files are required for deployment.

* crd.yaml
* operator.yaml
* obcluster.yaml

2, Modify configuration files before deployment.

a. configuration operator.yaml

The --cluster-name startup parameter is recommended to be the same as the Kubernetes cluster name.

b. Configure node label.

The Kubernetes nodes need to be labeled with labels that match nodeSelector configuration in obcluster.yaml. Ob-operator 
will schedule the Pod to the node with the corresponding label.You are advised to set the key of the label to `topology.kubernetes.io/zone`
you can use label plugin.For example label.yaml:

```shell
apiVersion: sealer.aliyun.com/v1alpha1
kind: Plugin
metadata:
  name: MyLabel
spec:
  type: LABEL
  action: PreGuest
  data: |
    192.168.0.2 topology.kubernetes.io/zone=zonename
```

3, kubefile context

Kubefile：

```shell
FROM kubernetes:v1.19.8
COPY openebs-operator.yaml etc/
COPY cstor-operator.yaml etc/
COPY crd.yaml etc/
COPY operator.yaml etc/
COPY obcluster.yaml etc/
COPY label.yaml plugins
COPY imageList manifests
CMD kubectl apply -f etc/openebs-operator.yaml
CMD kubectl apply -f etc/cstor-operator.yaml
CMD kubectl apply -f etc/crd.yaml
CMD kubectl apply -f etc/operator.yaml
CMD kubectl apply -f etc/obcluster.yaml
```

## How to build it

```shell
sealer build -f Kubefile -t {Your Image Name} .
```