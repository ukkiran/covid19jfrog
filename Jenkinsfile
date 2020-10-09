pipeline {
    environment {
                registry = "ukkb96/covid19"
                registryCredential = 'dockerhub'
                dockerImage = ''
                }
     agent { label 'master' }
        stages {
           stage('git checkout') {
            steps{
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'github', url: 'https://github.com/ukkiran/covid19.git']]])
            }
        }
        
          stage('build and package ') {
            steps{
               sh 'mvn clean package'
            }
         }
         stage('test') {
            steps{
                sh 'mvn test'
            }
         }
               
         stage('Build image') {
            steps{
                script {
                   docker.build registry + ":$BUILD_NUMBER"
                }
            }
         }
         
         stage('push image') {
            steps{
                script{
                   docker.withRegistry( '', registryCredential ) {
                    sh 'docker push ukkb96/covid19":$BUILD_NUMBER"'
                  
                    }
                }
            }
         }
          stage('run application as an docker container on tomcat') {
            steps{
                sh 'docker run -dt --name covid19 -p 8085:8080 ukkb96/covid19":$BUILD_NUMBER"'
            }
         }
           stage('run codecoverage on sonarqube server') {
            steps{
                sh 'mvn sonar:sonar \
                     -Dsonar.projectKey=covid19sonarqube \
                     -Dsonar.host.url=http://localhost:9001 \
                     -Dsonar.login=4cd6abc8a4287b96a6952eb6792c6815c867c5c0'
                }
            }
            stage('uploading artifacts to jfrog artifactory') {
            steps{
                script{
                    sh 'cp ./settings.xml ./m2/'
                    sh 'cp ./settings-security.xml ./m2/'
                    sh 'mvn clean deploy'
            }
          }
    }
}
