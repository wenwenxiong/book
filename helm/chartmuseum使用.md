### chartmusuem
Install local executable file:    

```
# curl -LO https://s3.amazonaws.com/chartmuseum/release/latest/bin/linux/amd64/chartmuseum
# chmod +x ./chartmuseum
# mv ./chartmuseum /usr/local/bin
```
Initialize with the local filesystem storage:    

```
# sudo mkdir chartstorage
# sudo chartmuseum --debug --port=8988 --storage="local" --storage-local-rootdir="./chartstorage"
```
Now visit your `http://localhost:8988` you could reach the chartmusuem.    

Using chartmusuem:    

```
# helm repo add chartmuseum http://localhost:8988
# helm update
```
Upload chart:    

```
# git clone https://github.com/stakater/chart-mysql.git
# cd chart-mysql
# cd mysql
# helm lint
# helm package .
# curl -L --data-binary "@mysql-1.0.1.tgz" http://localhost:8988/api/charts
```