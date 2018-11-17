# airflow-on-k8s
在k8s集群上构建airflow
官方给的Airflow构建于K8s集群不太详细，并且有一些坑，特此说明

## Docker构建及加速

在/software/airflow/incubator-airflow/scripts/ci/kubernetes/docker/下执行build.sh即可自动构建airflow的镜像
该镜像使用ubantu基础镜像，依赖国外apt源以及pip源，本项目在Dockerfile中都修改为了国内的阿里源，记得把.pip目录拷贝到build.sh同目录下。

## 修改基于集群的Volume

在构建好镜像之后，执行/software/airflow/incubator-airflow/scripts/ci/kubernetes/kube/的deploy.sh即可自动完成部署，但是对于非minikube集群，会有以下几个坑：

### 1 单节点kubernetes集群在master上无法部署airflow的Pod

需要修改yaml使其支持，具体请参考官方关于taint的介绍

### 2 集群状态的Volume修改

在部署过程中需要生成3个pv和pvc，都是基于hostPath的，hostPath在集群情况下并不能有效工作，在volumes.yaml中将其修改为特定集群支持的存储方式，我这里使用nfs。

### 3 集群状态的accessModes修改

官方给定的是其中的airflow-tags和test-volume两个pv使用ReadWriteOnce但是pvc使用ReadWriteMany导致无法被Bound，我这里统一修改为ReadWriteMany就ok了。

另外打好的镜像名为airflow，需要修改为docker.io/airflow

  docker tag airflow docker.io/airflow
