### 修改点
1、`kubernetes`版本为`1.10.0 `，`ansible`中的脚本正则表达式有问题
原来的脚本中为`main.yaml`的`22`行左右的`v([[:digit:]]\.[[:digit:]]\.[[:digit:]])`修改为`v([[:digit:]]\.[[:digit:]]{1,}\.[[:digit:]])`
```
- name: Extract server version
  shell: |
    {{ cmd_path }} version | sed -En "{{'s/kubernetes.*v([[:digit:]]\.[[:digit:]]\.[[:digit:]]).*/\1/p' if cluster_flavour == 'ocp' else 's/Server Version.*GitVersion.*v([[:digit:]]\.[[:digit:]]\.[[:digit:]]).*/\1/p'}}" | tail -1
  register: vo
```
修改为
```
- name: Extract server version
  shell: |
    {{ cmd_path }} version | sed -En "{{'s/kubernetes.*v([[:digit:]]\.[[:digit:]]{1,}\.[[:digit:]]).*/\1/p' if cluster_flavour == 'ocp' else 's/Server Version.*GitVersion.*v([[:digit:]]\.[[:digit:]]{1,}\.[[:digit:]]).*/\1/p'}}" | tail -1
  register: vo
```
2、ansible脚本配置获取istio安装yaml文件

脚本中的逻辑是首先检测目标机器的~/.istio目录下是否存在指定版本的istio文件夹存在，例如0.7.1版本，则在目标机器检测目录
```
~/.istio/istio-0.7.1
```
是否存在。如果没有检测目标机器的~/.istio/istio-0.7.1文件夹存在。则会到github上下载解压。
无internet网络连接时，可以手动的拷贝istio-0.7.1文件夹到目标机器的~/.istio/istio-0.7.1。在install_distro.yml文件中加入了以下脚本把ansible目录下的istio-0.7.1.tar.gz文件解压到目标机器的/root/.istio目录下。
```
- name: untar local istio tar to remote dir
  unarchive:
    src: istio-0.7.1.tar.gz
    dest: /root/.istio
```

3、增添脚本在目标机器配置istio自动注入功能
在ansible脚本安装完addons后加入一个脚本install_sidecar.yml，其内容为
```
- name: copy istio proxy images to remote
  copy:
    src: istioproxy.tar.xz
    dest: /root/istioproxy.tar.xz

- name: copy istio proxy init images to remote
  copy:
    src: istioproxyinit.tar.xz
    dest: /root/istioproxyinit.tar.xz

- name: load proxy and proxy init image
  shell: |
    docker load -i /root/istioproxy.tar.xz && docker load -i /root/istioproxyinit.tar.xz

- name: Install webhook
  shell: |
    {{ istio_dir }}/install/kubernetes/webhook-create-signed-cert.sh --service istio-sidecar-injector --namespace istio-system --secret sidecar-injector-certs

- name: install sidecar configmap
  shell: |
    {{ cmd_path }} apply -f {{ istio_dir }}/install/kubernetes/istio-sidecar-injector-configmap-release.yaml

- name: config caBundle
  shell: |
    cat {{ istio_dir }}/install/kubernetes/istio-sidecar-injector.yaml | {{ istio_dir }}/install/kubernetes/webhook-patch-ca-bundle.sh > {{ istio_dir }}/install/kubernetes/istio-sidecar-injector-with-ca-bundle.yaml

- name: Install sidecar injector webhook on Kubernetes
  shell: |
    {{ cmd_path }}  apply -f {{ istio_dir }}/install/kubernetes/istio-sidecar-injector-with-ca-bundle.yaml
```
这里只是把镜像导入到目标机器中，实际环境还要考虑其他工作节点的镜像导入。
install_sidecar.yml在main.yaml的位置为
```
- include_tasks: install_on_cluster.yml
- include_tasks: install_sidecar.yml
- include_tasks: install_samples.yml
  when: (istio.samples is iterable) and (istio.samples | length > 0)
```
### 整合到kismatic中
kismatic中的命令行安装有以下命令可以执行单个在ansible/playbooks目录下的yaml文件

```
./kismatic install step _istio.yaml
```
上面的命令单独执行ansible/playbooks/_istio.yaml文件，遇到问题
```
ansible/bin/ansible-playbook -i ansible/inventory.ini -s ansible/playbooks/_istio.yaml --extra-vars @ansible/clustercatalog.yaml -vvvv
2018-05-04 04:32:21.784+0000 - Using ansible/playbooks/ansible.cfg as config file
2018-05-04 04:32:21.913+0000 - ERROR! no action detected in task. This often indicates a misspelled module name, or incorrect module path.
2018-05-04 04:32:21.913+0000 - 
2018-05-04 04:32:21.913+0000 - The error appears to have been in '/root/deploy/cluster00/ansible/playbooks/roles/istio/tasks/main.yml': line 7, column 3, but may
2018-05-04 04:32:21.913+0000 - be elsewhere in the file depending on the exact syntax problem.
2018-05-04 04:32:21.913+0000 - 
2018-05-04 04:32:21.913+0000 - The offending line appears to be:
2018-05-04 04:32:21.913+0000 - 
2018-05-04 04:32:21.913+0000 - 
2018-05-04 04:32:21.913+0000 - - include_tasks: set_implicitly_disabled_addons.yml
2018-05-04 04:32:21.913+0000 -   ^ here
```
我的ansible-playbook版本是2.5.1
但是，越过kismatic，执行使用ansible-playbook命令执行，可以成功
```
ansible/bin/ansible-playbook -i ansible/inventory.ini -s ansible/playbooks/_istio.yaml --extra-vars @ansible/clustercatalog.yaml -vvvv
```