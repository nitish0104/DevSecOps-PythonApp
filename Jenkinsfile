pipeline {
    agent any
    tools {
        jdk 'JDK'  
        nodejs 'NodeJs'
    }
    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
        SCANNER_HOME = tool 'sonar-scanner'
        registryCredential = 'Docker'
        JOB_NAME= "${JOB_BASE_NAME}"
        GITHUB_TOKEN = "${Github}"
        BUILD_URL= "${BUILD_URL}"

    }
    stages {    
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                script {
                    git credentialsId: 'Github', 
                        url: 'https://github.com/nitish0104/DevSecOps-Python-Landingpage.git',
                        branch: 'master'
                }
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('Sonarqube-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Landingpage \
                    -Dsonar.projectKey=Landingpage'''
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonarqube'
                }
            }
        }
        
        stage('TRIVY FS SCAN') {
             steps {
                 sh "trivy fs . > trivyfs.txt"
             }
        }
        stage('Build Docker'){
            steps{
                script{
                        sh '''
                        echo 'Build Docker Image'
                        docker build -t nitish0104/devsecops-python:${BUILD_NUMBER} .
                        echo 'Docker Build Completed'
                        '''
                    }
            }
        }

        stage('Push Docker image to dockerHub'){
           steps{
                script{
                    docker.withRegistry('', registryCredential) {
                        sh '''
                        echo 'Logging into Docker'
                        docker login
                        echo 'Push to Docker  Repo'
                        docker push nitish0104/devsecops-python:${BUILD_NUMBER}
                        '''
                    }
                }
            }
        }
        stage("TRIVY Image Scan"){
            steps{
                sh "trivy image nitish0104/devsecops-python:${BUILD_NUMBER} > trivyimage.txt" 
            }
        }
        stage("CI Done"){
            steps{
                echo "Done CI pipeline Done"
            }      
        }
        stage('Checkout K8S manifest SCM'){
            steps {
                git credentialsId: 'Github', 
                url: 'https://github.com/nitish0104/DevSecOps-landingpage-mainfest.git',
                branch: 'master'
            }
        }
        stage('Update K8S manifest & push to Repo'){
            steps {
                script{
                    withCredentials([usernamePassword(credentialsId: 'Github', passwordVariable: 'GITHUB_TOKEN', usernameVariable: 'nitish0104')]) {
                        sh '''
                        cat Deployment.yaml
                        sed -i "s|8|${BUILD_NUMBER}|g" Deployment.yaml
                        cat Deployment.yaml
                        git config --global user.email "nitishdalvi1@gmail.com"
                        git config --global user.name "nitish0104"
                        echo "git configuration done"
                        git remote set-url origin https://nitish0104:${GITHUB_TOKEN}@github.com/nitish0104/DevSecOps-landingpage-mainfest.git
                        git add Deployment.yaml
                        git commit -m 'Updated the Deployment yaml | Jenkins Pipeline'
                        git push origin HEAD:master
                        '''                        
                    }
                }
            }
        }
        stage("CD Done"){
            steps{
                echo "Done CI pipeline Done"
            }      
        }
        
    }


}