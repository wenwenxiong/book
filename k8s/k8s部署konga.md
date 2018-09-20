###k8s部署konga
konga的数据库可以使用mongodb，postgres，mysql，sql server。默认使用mongodb作为数据库，接下来分别使用mongodb和postgres数据库来部署konga到kubernetes环境。
####konga使用mongodb作为数据库
首先，构建mongodb数据库。mongdb的三个yaml文件为```mongo-data-persistentvolumeclaim.yaml```,```mongo-deployment.yaml```,```mongo-service.yaml```。
mongo-data-persistentvolumeclaim.yaml为创建一个mongdb的一个持久化存储pvc卷。
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  creationTimestamp: null
  labels:
    io.kompose.service: mongo-data
  name: mongo-data
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
status: {}
```
使用的是kubernetes环境的默认存储storageclass。
mongdb的deploymemt和service的yaml文件如下：
mongo-deployment.yaml
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    kompose.cmd: ./kompose -f docker-compose.yaml convert
    kompose.version: 1.14.0 (fa706f2)
  creationTimestamp: null
  labels:
    io.kompose.service: mongo
  name: mongo
spec:
  replicas: 1
  strategy:
    type: Recreate
  template:
    metadata:
      creationTimestamp: null
      labels:
        io.kompose.service: mongo
    spec:
      containers:
      - image: mongo
        imagePullPolicy: IfNotPresent
        name: mongo
        ports:
        - containerPort: 27017
          hostIP: 127.0.0.1
        resources: {}
        volumeMounts:
        - mountPath: /data/db
          name: mongo-data
      restartPolicy: Always
      volumes:
      - name: mongo-data
        persistentVolumeClaim:
          claimName: mongo-data
status: {}
```
mongo-service.yaml
```
apiVersion: v1
kind: Service
metadata:
  annotations:
    kompose.cmd: ./kompose -f docker-compose.yaml convert
    kompose.version: 1.14.0 (fa706f2)
  creationTimestamp: null
  labels:
    io.kompose.service: mongo
  name: mongo
spec:
  ports:
  - name: "27017"
    port: 27017
    targetPort: 27017
  selector:
    io.kompose.service: mongo
status:
  loadBalancer: {}
```
创建konga的deployment和service的yaml文件如下，
konga-deployment.yaml
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
  creationTimestamp: null
  labels:
    io.kompose.service: konga
  name: konga
spec:
  replicas: 1
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        io.kompose.service: konga
    spec:
      containers:
      - env:
        - name: DB_ADAPTER
          value: mongo
        - name: DB_DATABASE
          value: konga
        - name: DB_HOST
          value: mongo
        - name: NODE_ENV
          value: development
        image: pantsel/konga:latest
        imagePullPolicy: IfNotPresent
        name: konga
        ports:
        - containerPort: 1337
        resources: {}
      restartPolicy: Always
status: {}
```
在环境变量的配置中，设置了konga的DB_ADAPTER为mongo，设置了数据库名称为konga。
konga-service.yaml
```
apiVersion: v1
kind: Service
metadata:
  annotations:
    kompose.cmd: ./kompose -f docker-compose.yaml convert
    kompose.version: 1.14.0 (fa706f2)
  creationTimestamp: null
  labels:
    io.kompose.service: konga
  name: konga
spec:
  ports:
  - name: "1337"
    port: 1337
    targetPort: 1337
  selector:
    io.kompose.service: konga
status:
  loadBalancer: {}
```
####konga使用postgres作为数据库
kong的数据库可以使用cassandra、postgres数据库，konga也可以使用postgres数据库，在部署中，可以在一个postgres实例中创建两个数据库，一个供kong使用，一个供konga使用。
创建postgres的yaml文件如下
```
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  ports:
  - name: pgql
    port: 5432
    targetPort: 5432
    protocol: TCP
  selector:
    app: postgres

---
apiVersion: v1
kind: ReplicationController
metadata:
  name: postgres
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:9.6
          env:
            - name: POSTGRES_USER
              value: kong
            - name: POSTGRES_PASSWORD
              value: kong
            - name: POSTGRES_DB
              value: kong
            - name: PGDATA
              value: /var/lib/postgresql/data/pgdata
          ports:
            - containerPort: 5432
          volumeMounts:
            - mountPath: /var/lib/postgresql/data
              name: pg-data
      volumes:
        - name: pg-data
          emptyDir: {}
```
这里postgres数据库的存储使用的是临时存储，也就是一旦pod删除，存储也就消失。用户可以根据环境配置自己的存储。
创建一个job来创建数据库名为```konga_database```供konga使用（konga的代码中默认设置konga_database为数据库名称，可以通过配置环境变量来自定义数据库的名称）。
postgres_kongadb.yaml
```
apiVersion: batch/v1
kind: Job
metadata:
  name: konga-db-create
spec:
  template:
    metadata:
      name: konga-db-create
    spec:
      containers:
      - name: konga-db-create
        image: postgres:9.4
        env:
          - name: POSTGRES_USER
            value: kong
          - name: POSTGRES_PASSWORD
            value: kong
          - name: POSTGRES_DB
            value: kong
        command: [ "/bin/sh", "-c", "PGPASSWORD=kong psql -h postgres -U kong -W kong -c 'CREATE DATABASE konga_database'" ]
      restartPolicy: Never
```
创建konga。
```
apiVersion: v1
kind: Service
metadata:
  name: konga
spec:
  type: NodePort
  ports:
  - name: konga
    port: 1337
    targetPort: 1337
    protocol: TCP
  selector:
    app: konga
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: konga
spec:
  replicas: 1
  template:
    metadata:
      labels:
        name: konga
        app: konga
    spec:
      containers:
      - name: konga
        image: pantsel/konga
        env:
          - name: DB_ADAPTER
            value: 'postgres'
          - name: DB_HOST
            value: postgres
          - name: DB_USER
            value: kong
          - name: DB_PASSWORD
            value: kong
        ports:
        - name: konga
          containerPort: 1337
          protocol: TCP
```
在环境变量的配置中，设置了konga的DB_ADAPTER为postgres，设置了数据库主机名为postgres，也是kuberntes中创建postgres是的主机名。