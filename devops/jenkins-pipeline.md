新建jenkins pipeline项目
![jenkins-pipeline](./images/jenkins-pipeline.png "jenkins-pipeline")
配置jenkins pipeline项目
![jenkins-pipeline2](./images/jenkins-pipeline2.png "jenkins-pipeline2")

jenkins安装在非容器环境，可以使用docker-compose命令,能成功执行Pipeline script。
jenkins安装在容器环境（），发现即使把docker-compose安装到jenkins容器中，但是容器中运行docker-compose报错如下
```
bash-4.3# /usr/bin/docker-compose --version
bash: /usr/bin/docker-compose: No such file or directory
```
Pipeline script
```
node{
    stage('git clone'){
        //check out
        git credentialsId: 'test', url: 'http://172.17.0.4/root/test.git'
    }
    
    stage('build'){
        docker.image('192.168.122.1:5000/mymaven:3.5-jdk8').inside{
            sh 'mvn package -B -DskipTests'
        }
    }
    
    stage('image build and push'){
        docker.withRegistry('http://192.168.122.1:5000', 'dockerregistry'){
            docker.build('piggymetrics/config:${BUILD_ID}', '-f config/Dockerfile ./config').push()
        }
        docker.withRegistry('http://192.168.122.1:5000', 'dockerregistry'){
            docker.build('piggymetrics/auth-service:${BUILD_ID}', '-f auth-service/Dockerfile ./auth-service').push()
        }
        docker.withRegistry('http://192.168.122.1:5000', 'dockerregistry'){
            docker.build('piggymetrics/account-service:${BUILD_ID}', '-f account-service/Dockerfile ./account-service').push()
        }
        docker.withRegistry('http://192.168.122.1:5000', 'dockerregistry'){
            docker.build('piggymetrics/gateway:${BUILD_ID}', '-f gateway/Dockerfile ./gateway').push()
        }
        docker.withRegistry('http://192.168.122.1:5000', 'dockerregistry'){
            docker.build('piggymetrics/mongodb:${BUILD_ID}', '-f mongodb/Dockerfile ./mongodb').push()
        }
        docker.withRegistry('http://192.168.122.1:5000', 'dockerregistry'){
            docker.build('piggymetrics/monitoring:${BUILD_ID}', '-f monitoring/Dockerfile ./monitoring').push()
        }
        docker.withRegistry('http://192.168.122.1:5000', 'dockerregistry'){
            docker.build('piggymetrics/notification-service:${BUILD_ID}', '-f notification-service/Dockerfile ./notification-service').push()
        }
        docker.withRegistry('http://192.168.122.1:5000', 'dockerregistry'){
            docker.build('piggymetrics/statistics-service:${BUILD_ID}', '-f statistics-service/Dockerfile ./statistics-service').push()
        }
        docker.withRegistry('http://192.168.122.1:5000', 'dockerregistry'){
            docker.build('piggymetrics/registry:${BUILD_ID}', '-f registry/Dockerfile ./registry').push()
        }
    }
    
    stage('deploy images'){
        sh 'docker-compose down'
        sh 'export CONFIG_SERVICE_PASSWORD="engine123"'
        sh 'export NOTIFICATION_SERVICE_PASSWORD="engine123"'
        sh 'export STATISTICS_SERVICE_PASSWORD="engine123"'
        sh 'export ACCOUNT_SERVICE_PASSWORD="engine123"'
        sh 'MONGODB_PASSWORD="engine123"'
        sh 'printenv'
        sh 'cp docker-compose.yml.template docker-compose.yml'
        sh 'sed -i "s#REPLACE#${CI_COMMIT_SHA:0:7}#g" docker-compose.yml'
        sh 'docker-compose up -d'
    }
}
```