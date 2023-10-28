pipeline {
    agent any
     tools {
  maven 'M2_HOME'
}
environment {
    registry = '216174199642.dkr.ecr.us-east-1.amazonaws.com/jenkins_ecr'
    registryCredential = 'AWS_ECR_ID'
    dockerimage = ''

     NEXUS_VERSION = "nexus3"
     NEXUS_PROTOCOL = "http"
     NEXUS_URL = "45.56.79.252:8081"
     NEXUS_REPOSITORY = "biomed"
     NEXUS_CREDENTIAL_ID = "Nexus_2nd"
     POM_VERSION = ''
}
    stages {

        // stage("build & SonarQube analysis") {  
        //     steps {
        //         echo 'build & SonarQube analysis...'
        //        withSonarQubeEnv('SonarServer') {
        //            sh 'mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=McGarh_geolocation -X'
        //        }
        //     }
        //   }
        // stage('Check Quality Gate') {
        //     steps {
        //         echo 'Checking quality gate...'
        //          script {
        //              timeout(time: 20, unit: 'MINUTES') {
        //                  def qg = waitForQualityGate()
        //                  if (qg.status != 'OK') {
        //                      error "Pipeline stopped because of quality gate status: ${qg.status}"
        //                  }
        //              }
        //          }
        //     }
        // }
        
         
        stage('maven package') {
            steps {
                sh 'mvn clean'
                sh 'mvn package -DskipTests'
            }
        }
        stage('Build Image') {
            
            steps {
                script{
                  def mavenPom = readMavenPom file: 'pom.xml'
                  POM_VERSION = "${mavenPom.version}"
                  echo "${POM_VERSION}"
                  dockerImage = docker.build registry + ":${POM_VERSION}"
                } 
            }
        }
        stage('push image') {
            steps{
                script{ 
                    docker.withRegistry("https://"+registry,"ecr:us-east-1:"+registryCredential) {
                        dockerImage.push()
                    }
                }
            }
        } 

        // Project Helm Chart push as tgz file
        stage("pushing the Backend helm charts to nexus"){
            steps{
                script{
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId:'Nexus_2nd', usernameVariable: 'admin', passwordVariable: 'student1']]) {
                            def mavenPom = readMavenPom file: 'pom.xml'
                            POM_VERSION = "${mavenPom.version}"
                            sh "echo ${POM_VERSION}"
                            sh "tar -czvf  app-${POM_VERSION}.tgz app/"
                            sh "curl -u admin:$student1 http://45.56.79.252:8081/repository/biomed/ --upload-file app-${POM_VERSION}.tgz -v"  
                    }
                } 
            }
        }
    }     	    
}