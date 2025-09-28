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
                echo "VITE_API_URL = "http://98.84.107.220:8081/api"" > .env
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



# 3 tier application deployment using docker with jenkins

```groovy
pipeline {
    agent any
    environment {
        SPRING_DATASOURCE_URL = "jdbc:mariadb://mydb.cujwsmiewj2i.us-east-1.rds.amazonaws.com:3306/student_db"
        SPRING_DATASOURCE_USERNAME = "admin"
        SPRING_DATASOURCE_PASSWORD = "Rajesh1234"
        VITE_API_URL = "http://98.84.107.220:8081/api"
    }
    stages{
        stage('clone reposirory'){
            steps{
                git branch: 'main', url: 'https://github.com/RajeshGajengi/EasyCRUD-K8s.git'
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
                sh '''
                cd backend
                docker run -d -p 8080:8081 r25gajengi/easy_backend:v1
                '''
            }
        }
        stage('Build Frontend image adn Push to Docker Hub'){
            steps{
                sh '''
                docker build -t r25gajengi/easy_frontend:v1
                docker push r25gajengi/easy_frontend:v1
                '''
            }
        }
        stage('Run frontend container'){
            steps{
                sh '''
                cd frontend
                docker run -d -p 80:80 easy_frontend:v1
                '''
            }
        }
    }
}

```