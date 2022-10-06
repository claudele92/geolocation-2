
pipeline {
    triggers {
  pollSCM('* * * * *')
    }
   agent any
    tools {
  maven 'M2_HOME'
}
environment {
    registry = '614938384999.dkr.ecr.us-east-1.amazonaws.com/geo2'
    registryCredential = 'jenkins-ecr'
    dockerimage = ''
}

    stages {

        stage("build & SonarQube analysis") {
            agent {
        docker { image 'maven:3.8.6-openjdk-11-slim' }
   }
            
            
            steps {
              withSonarQubeEnv('SonarServer') {
                 // sh 'mvn sonar:sonar -Dsonar.projectKey=kserge2001_geolocation -Dsonar.java.binaries=.'
                  sh 'mvn sonar:sonar -Dsonar.projectKey=geo-job-meeting -Dsonar.organization=geo-job-meeting-1 -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=272e5536e9efc850018f1af48bd1017e63bc76b9'
              }
            }
          }
        stage('Check Quality Gate') {
            steps {
                echo 'Checking quality gate...'
                script {
                    timeout(time: 20, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline stopped because of quality gate status: ${qg.status}"
                        }
                    }
                }
            }
        }
        
         
        stage('maven package') {
            steps {
                sh 'mvn clean'
                sh 'mvn install -DskipTests'
                sh 'mvn package -DskipTests'
            }
        }
        stage('Build Image') {
            
            steps {
                script{
                  def mavenPom = readMavenPom file: 'pom.xml'
                    dockerImage = docker.build registry + ":${mavenPom.version}"
                } 
            }
        }
        stage('Deploy image') {
           
            
            steps{
                script{ 
                    docker.withRegistry("https://"+registry,"ecr:us-east-1:"+registryCredential) {
                        dockerImage.push()
                    }
                }
            }
        }    
         
         
    }
}
