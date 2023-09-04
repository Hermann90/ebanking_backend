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
     NEXUS_URL = "http://ec2-18-237-195-147.us-west-2.compute.amazonaws.com"
     NEXUS_REPOSITORY = "utrains-nexus-pipeline"
     NEXUS_CREDENTIAL_ID = "nexus-user-credentials"
     POM_VERSION = ''
}


    stages {
         stage('conf ENV') {
            steps {
                script{
                    echo "${NEXUS_URL}:8081/repository/custom_scripts/devops_utils/init_env.sh"
                    sh "sudo /home/ec2-user/ebanking_backend/init_env.sh"
                    echo "test: ${NEXUS_USER}"
                    sh """#!/bin/bash

                    echo curl -L -u $NEXUS_USER:$NEXUS_PASSWORD -X GET "$NEXUS_URL:8081/repository/custom_scripts/devops_utils/init_env.sh" -H "accept: application/json" -o init_env.sh
                    echo curl -L -u ${NEXUS_USER}:${NEXUS_PASSWORD} -X GET "$NEXUS_URL:8081/repository/custom_scripts/devops_utils/conf_nexus_repo.xml" -H "accept: application/json" -o conf_nexus_repo.xml
                    
                    curl -L -u $NEXUS_USER:$NEXUS_PASSWORD -X GET "$NEXUS_URL:8081/repository/custom_scripts/devops_utils/init_env.sh" -H "accept: application/json" -o init_env.sh
                    curl -L -u $NEXUS_USER:$NEXUS_PASSWORD -X GET "$NEXUS_URL:8081/repository/custom_scripts/devops_utils/conf_nexus_repo.xml" -H "accept: application/json" -o conf_nexus_repo.xml
                    chmod +x init_env.sh
                    ls
                    cat init_env.sh
                    ./init_env.sh
                    printenv
                    """
                }
            }
        }

        /*stage('maven package') {
            steps {
                script{
                    sh '''sudo cat conf_nexus_repo.xml > /opt/maven/conf/settings.xml
                    mvn clean
                    mvn package -DskipTests
                    echo http://${NEXUS_URL}:8081/repository/custom_scripts/devops_utils/init_env.sh
                    '''
                }
            }
        }
        stage('Hello') {
            steps{
                script{
                  output= mul()
                  echo "The mul is ${output}"
                  echo $NEXUS_URL
                  sh ''' 
                    sudo cat /opt/maven/conf/settings.xml
                    export REPO_URL=http://ec2-18-237-195-147.us-west-2.compute.amazonaws.com:8081/repository/devops_utils/
                    export REPO_ID=devops_utils

                    echo "The mul is ${output}"
                    'echo  the version is : ${output}'
                    mvn deploy:deploy-file \
                    -Durl=$REPO_URL \
                    -DrepositoryId=$REPO_ID \
                    -DgroupId=org.sid \
                    -DartifactId=ebanking-backend \
                    -Dversion=0.0.1-SNAPSHOT  \
                    -Dpackaging=jar \
                    -Dfile=target/ebanking-backend-0.0.1-SNAPSHOT.jar
                  '''
            }
            }
        }*/
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

