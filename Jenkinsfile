def mul()
{
    def pom = readMavenPom file: "pom.xml";
   def a=2
   def b=3
   def mul=a*b
   return pom.version
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
         stage('conf ENV') {
            steps {
                script{
                    echo "http://${NEXUS_URL}:8081/repository/custom_scripts/devops_utils/init_env.sh"
                    sh '''#!/bin/bash
                    #wget "http://${NEXUS_URL}/repository/custom_scripts/devops_utils/init_env.sh"
                    curl -L -u ${NEXUS_USER}:${NEXUS_PASSWORD} -X GET "${NEXUS_URL}:8081/repository/custom_scripts/devops_utils/init_env.sh" -H "accept: application/json" -o init_env.sh
                    chmod +x init_env.sh
                    ls
                    ./init_env.sh
                    printenv
                    '''
                }
            }
        }

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
                  sh '''
                    export REPO_URL=http://ec2-18-237-195-147.us-west-2.compute.amazonaws.com:8081/repository/devops_utils/
                    export REPO_ID=devops_utils
                    mvn deploy:deploy-file \
                    -Durl=$REPO_URL \
                    -DrepositoryId=$REPO_ID \
                    -DgroupId=org.sid \
                    -DartifactId=ebanking-backend \
                    -Dinternal.repo.username=admin \
                    -Dinternal.repo.password=devops \
                    -Dversion=0.0.1-SNAPSHOT  \
                    -Dpackaging=jar \
                    -Dfile=target/ebanking-backend-0.0.1-SNAPSHOT.jar
                  '''
            }
            }
        }
       /* stage('Build Image') {
            steps {
                 script{
                    def mavenPom = readMavenPom file: 'pom.xml'
                    POM_VERSION = "${mavenPom.version}"
                    echo "${POM_VERSION}"
                    dockerImage = docker.build registry + ":${POM_VERSION}"
                }
            }
        }*/
    }
}

