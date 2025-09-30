#  3 tier application deployement in local Server with jenkins


## Option 1 : 
```groovy
pipeline {
    agent any
    environment {
        SPRING_DATASOURCE_URL = "jdbc:mariadb://mydb.cujwsmiewj2i.us-east-1.rds.amazonaws.com:3306/student_db"
        SPRING_DATASOURCE_USERNAME = "admin"
        SPRING_DATASOURCE_PASSWORD = "Rajesh1234"
    }
    stages{
        stage('clone reposirory'){
            steps{
                git branch: 'main', url: 'https://github.com/RajeshGajengi/EasyCRUD-K8s.git'
            }
        }
        stage('backend build'){
            steps{
                sh '''
                cd backend
                mvn clean package
                '''
            }
        }
        stage('frontend build'){
            steps{
                sh '''
                cd frontend
                echo "VITE_API_URL = "http://54.235.2.116:8081/api"" > .env
                npm install
                npm run build
                '''
            }
        }
        stage('frontend deploy'){
            steps{
                sh '''
                cd frontend
                sudo cp -rf dist/* /var/www/html/
                '''
            }
        }
        stage('backend deploy'){
            steps{
                sh '''
                cd backend
                java -jar target/student-registration-backend-0.0.1-SNAPSHOT.jar
                '''
            }
        }
    }
}

```
Option 2 :
```groovy

pipeline {
    agent any
    environment {
        SPRING_DATASOURCE_URL = "jdbc:mariadb://mydb.cujwsmiewj2i.us-east-1.rds.amazonaws.com:3306/student_db"
        SPRING_DATASOURCE_USERNAME = "admin"
        SPRING_DATASOURCE_PASSWORD = "Rajesh1234"
        VITE_API_URL = "http://18.207.155.254:8081/api"
    }
    stages{
        stage('clone reposirory'){
            steps{
                git branch: 'main', url: 'https://github.com/RajeshGajengi/EasyCRUD-K8s.git'
            }
        }
        stage('backend build'){
            steps{
                sh '''
                cd backend
                mvn clean package
                '''
            }
        }
        stage('frontend build'){
            steps{
                sh '''
                cd frontend
                npm install
                npm run build
                '''
            }
        }
        stage('frontend deploy'){
            steps{
                sh '''
                cd frontend
                sudo cp -rf dist/* /var/www/html/
                '''
            }
        }
        stage('backend deploy'){
            steps{
                sh '''
                cd backend
                java -jar target/student-registration-backend-0.0.1-SNAPSHOT.jar
                '''
            }
        }
    }
}

```

## Option 3 : With Jenkins credentials adding environments variables.

```groovy

pipeline {
    agent any
    stages{
        stage('clone reposirory'){
            steps{
                git branch: 'main', url: 'https://github.com/RajeshGajengi/EasyCRUD-K8s.git'
            }
        }
        stage('backend build'){
            steps{
                withCredentials([string(credentialsId: 'Database_url', variable: 'SPRING_DATASOURCE_URL'), string(credentialsId: 'Database_username', variable: 'SPRING_DATASOURCE_USERNAME'), string(credentialsId: 'DB_User_password', variable: 'SPRING_DATASOURCE_PASSWORD')]) {
                    sh '''
                cd backend
                mvn clean package
                '''
                }
            }
        }
        stage('frontend build'){
            steps{
                withCredentials([string(credentialsId: 'Backend_api', variable: 'VITE_API_URL')]) {
                     sh '''
                cd frontend
                npm install
                npm run build
                '''
                }
            }
        }
        stage('frontend deploy'){
            steps{
                withCredentials([string(credentialsId: 'Backend_api', variable: 'VITE_API_URL')]) {
                   sh '''
                cd frontend
                sudo cp -rf dist/* /var/www/html/
                '''
                }
            }
        }
        stage('backend deploy'){
            steps{
               withCredentials([string(credentialsId: 'Database_url', variable: 'SPRING_DATASOURCE_URL'), string(credentialsId: 'Database_username', variable: 'SPRING_DATASOURCE_USERNAME'), string(credentialsId: 'DB_User_password', variable: 'SPRING_DATASOURCE_PASSWORD')]) {
                    sh '''
                cd backend
                java -jar target/student-registration-backend-0.0.1-SNAPSHOT.jar
                '''
               }
            }
        }
    }
}

```

installation script :


#!/bin/bash
sudo apt update && apt install openjdk-17-jdk -y
sudo apt install maven -y
sudo apt install nodejs npm -y
sudo apt install mariadb-client -y
sudo apt install apache2 -y
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian/jenkins.io-2023.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins -y
sudo apt install docker.io -y




# 3 tier application deployment using docker with jenkins

sudo visudo
jenkins ALL=(ALL) NOPASSWD:ALL


Add Jenkins user to docker group
 - sudo usermod -aG docker jenkins
 - then restart the jenkins server

```groovy
pipeline {
    agent any
    stages{
        stage('clone reposirory'){
            steps{
                git branch: 'main', url: 'https://github.com/RajeshGajengi/EasyCRUD-K8s.git'
            }
        }
        stage('Docker login'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'docker-cred', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh 'docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}'
                }  
            }
        }
        stage('Build Backend image and push to Docker Hub'){
            steps{
               sh '''
                    cd backend
                    docker build -t r25gajengi/easy_backend:v1 .
                    docker push r25gajengi/easy_backend:v1
                    '''
            }
        }
        stage('Run Backend container'){
            steps{
               withCredentials([string(credentialsId: 'database_url', variable: 'SPRING_DATASOURCE_URL'), string(credentialsId: 'database_user', variable: 'SPRING_DATASOURCE_USERNAME'), string(credentialsId: 'database_password', variable: 'SPRING_DATASOURCE_PASSWORD')]) {
                sh '''
                cd backend
                docker rm -f backend || true
                docker run --name backend -d \
                  -p 8081:8081 \
                  -e SPRING_DATASOURCE_URL=${SPRING_DATASOURCE_URL} \
                  -e SPRING_DATASOURCE_USERNAME=${SPRING_DATASOURCE_USERNAME} \
                  -e SPRING_DATASOURCE_PASSWORD=${SPRING_DATASOURCE_PASSWORD} \
                  r25gajengi/easy_backend:v1
                '''
                }
            }
        }
        stage('Build Frontend image adn Push to Docker Hub'){
            steps{
                withCredentials([string(credentialsId: 'backend_api', variable: 'VITE_API_URL')]) {
                    sh '''
                    cd frontend
                    echo "VITE_API_URL = ${VITE_API_URL}" > .env
                    docker build -t r25gajengi/easy_frontend:v1 .
                    docker push r25gajengi/easy_frontend:v1
                    '''
                }
            }
        }
        stage('Run frontend container'){
            steps{
                sh '''
                cd frontend
                docker rm -f frontend || true
                docker run --name frontend -d \
                  -p 80:80 \
                  r25gajengi/easy_frontend:v1
                '''
            }
        }
    }
}


```


# 3 tier application deployment using k8s with jenkins

 ```groovy

 pipeline {
    agent any
    stages{
        stage('clone reposirory'){
            steps{
                git branch: 'main', url: 'https://github.com/RajeshGajengi/EasyCRUD-K8s.git'
            }
        }
        stage('Docker login'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'docker-cred', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh 'docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}'
                }  
            }
        }
        stage('Build Backend image and push to Docker Hub'){
            steps{
               sh '''
                    cd backend
                    docker build -t r25gajengi/easy_backend:v2 .
                    docker push r25gajengi/easy_backend:v2
                    '''
            }
        }
        
        stage('Build Frontend image adn Push to Docker Hub'){
            steps{
                sh '''
                    cd frontend
                    docker build -t r25gajengi/easy_frontend:v2 .
                    docker push r25gajengi/easy_frontend:v2
                    '''
            }
        }
        stage ('kubernetes configure'){
            steps{
                sh 'aws eks update-kubeconfig --region ap-south-1 --name mycluster'
            }
        }
        stage ('ingress controller download'){
            steps{
                sh 'kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml'
            }
        }
        stage ('Kubernetes deployment'){
            steps{
                sh '''
                kubectl apply -f k8s/secret.yaml
                kubectl apply -f k8s/backend-deployment.yaml
                kubectl apply -f k8s/backend-service.yaml
                kubectl apply -f k8s/frontend-deployment.yaml
                kubectl apply -f k8s/frontend-service.yaml
                kubectl apply -f k8s/ingress.yaml
                '''
            }
        }
        stage('verify'){
            steps{
                sh '''
                kubectl get pods
                kubectl get svc
                kubectl get ingress
                '''
            }
        }
    }
}

 

 ```