参考网址： https://github.com/kubernetes-sigs/kubespray/pull/3438/commits/58dc0c7479de8fbe026a3f0b2171d300f63e2597
https://kubernetes.io/docs/tasks/manage-gpus/scheduling-gpus/

###kubernetes调度NVIDIA GPU

前提
1）kubernetes节点预先安装了NVIDIA drivers
2）kubernetes节点预先安装了 nvidia-docker 2.0 deb或rpm包（https://github.com/nvidia/nvidia-docker/wiki/Installation-(version-2.0)#prerequisites）
3）配置docker的默认runtime为nvidia-container-runtime，具体做法是修改节点的docker配置文件```/etc/docker/daemon.json```，保证以下配置内容出现。
```
{
    "default-runtime": "nvidia",
    "runtimes": {
        "nvidia": {
            "path": "/usr/bin/nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}
```
4）在kubernetes上运行```nvidia-device-plugin```的```daemonset```。
```
kubectl create -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v1.11/nvidia-device-plugin.yml
```
5）测试，运行```gpu pod```测试效果。
```
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  containers:
    - name: cuda-container
      image: nvidia/cuda:9.0-devel
      resources:
        limits:
          nvidia.com/gpu: 2 # requesting 2 GPUs
    - name: digits-container
      image: nvidia/digits:6.0
      resources:
        limits:
          nvidia.com/gpu: 2 # requesting 2 GPUs
```

###kubespray中启用```gup```节点配置

Nvidia GPU Nodes
  ===============
  
   *Status: Experimental*
  
   Requirements
  ---------------------------
  
   Currently Ubuntu 16.04, Ubuntu 18.04 and Centos 7 are supported.
  
   Before you can succesfully install gpu scheduling in Kubernetes you need to make sure
  that the nouveau driver is disabled. There is a playbook `extra_playbooks/disable_nouveau.yml`
  which will disable nouveau and reboot your servers to make it permanent.
  
   Ofcourse you also need a set of Nvidia GPU cards. If you have multiple GPU models you should group.
  them by node. For more information on GPU scheduling in kubernetes follow this [link](https://kubernetes.io/docs/tasks/manage-gpus/scheduling-gpus/)
  
   Deployment
  --------------------------
  
   The driver installer requires the `overlay` storage system. In Centos this is default, in Ubuntu you should set in `all.yml`
  ```YAML
  docker_storage_options: -s overlay2
  ```
  
   In `k8s-cluster.yml` you should set the following:
  ```YAML
  nvidia_accelerator_enabled: true
  nvidia_gpu_nodes:
    - kube-gpu-001
    - kube-gpu-002
  nvidia_gpu_flavor: gtx
  ```
  After this you can configure other kubespray settings and run the `cluster.yml` playbook.
  
   Test deployment
  ------------
  
   Run the following pod to check your deployment
  
   ```YAML
  apiVersion: v1
  kind: Pod
  metadata:
    name: gpu-pod
  spec:
    tolerations:
        - key: "nvidia.com/gpu"
          effect: "NoSchedule"
          operator: "Exists"
    containers:
      - name: cuda-container
        image: nvidia/cuda
        command: ["/bin/sh"]
        args:
          - "-c"
          - for i in $(seq 1 1000); do sleep 20000 ; done
        resources:
          limits:
            nvidia.com/gpu: 1 # requesting 1 GPUs
  ```
  then run `kubectl exec gpu-pod -- nvidia-smi`. This should show your GPU info. If you requests 2 or more cards you should also see
  more gpu's with `nvidia-smi`.
  
   Notes
  ------------
  There is a `NoSchedule` setting on the nodes which contain GPU's. This will prevent non gpu pods to not be scheduled on a GPU node.