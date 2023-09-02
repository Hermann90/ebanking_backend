def mul()
{
   def a=2
   def b=3
   def mul=a*b
   return mul
}






pipeline {
    triggers {
  pollSCM('* * * * *')
    }
   agent any
    tools {
  maven 'M2_HOME'
}
environment {
    registry = '076892551558.dkr.ecr.us-east-1.amazonaws.com/jenkins'
    registryCredential = 'jenkins-ecr'
    dockerimage = ''

     NEXUS_VERSION = "nexus3"
     NEXUS_PROTOCOL = "http"
     NEXUS_URL = "139.177.192.139:8081"
     NEXUS_REPOSITORY = "utrains-nexus-pipeline"
     NEXUS_CREDENTIAL_ID = "nexus-user-credentials"
     POM_VERSION = ''
}


    stages {
        stage('maven package') {
            steps {
                script{
                    sh 'mvn clean'
                    sh 'mvn package -DskipTests'
                    echo "http://${NEXUS_URL}:8081/repository/custom_scripts/devops_utils/init_env.sh"
                }
            }
        }
        stage('Hello') {
            steps{
                script{
                  output= mul()
                  echo "The mul is ${output}"
            }
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
    }
}

