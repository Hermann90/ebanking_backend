def mul()
{
    def pom = readMavenPom file: "pom.xml";
   def a=2
   def b=3
   def mul=a*b
   return pom.version
}

def conf_python3_env(){
    def cm1 = "echo START ==============> install_and_config_python_modules"
    def cm2 = "apt update && sudo apt install python3-pip"

    sh(cm1,cm2)
}





pipeline {
    triggers {
  pollSCM('* * * * *')
    }
   agent any
    tools {
  maven 'M2_HOME'
}
/*environment {
    registry = '076892551558.dkr.ecr.us-east-1.amazonaws.com/jenkins'
    registryCredential = 'jenkins-ecr'
    dockerimage = ''

     NEXUS_VERSION = "nexus3"
     NEXUS_PROTOCOL = "http"
     NEXUS_URL = "http://ec2-18-237-195-147.us-west-2.compute.amazonaws.com"
     NEXUS_REPOSITORY = "utrains-nexus-pipeline"
     NEXUS_CREDENTIAL_ID = "nexus-user-credentials"
     POM_VERSION = ''
}*/


    stages {
         stage('conf ENV and make') {
            steps {
                script{
                    echo "$NEXUS_URL:8081/repository/$DEVOPS_SCRIPTS_REPO/init_env.sh"
                    sh "sudo /home/ec2-user/ebanking_backend/init_env.sh"
                    echo "test: $NEXUS_USER"
                    conf_python3_env()
                    sh """#!/bin/bash

                    echo curl -L -u $NEXUS_USER:$NEXUS_PASSWORD -X GET "$NEXUS_URL:8081/repository/$DEVOPS_SCRIPTS_REPO/init_env.sh" -H "accept: application/json" -o init_env.sh
                    echo curl -L -u $NEXUS_USER:$NEXUS_PASSWORD -X GET "$NEXUS_URL:8081/repository/$DEVOPS_SCRIPTS_REPO/conf_nexus_repo.xml" -H "accept: application/json" -o conf_nexus_repo.xml
                    
                    curl -L -u $NEXUS_USER:$NEXUS_PASSWORD -X GET "$NEXUS_URL:8081/repository/$DEVOPS_SCRIPTS_REPO/init_env.sh" -H "accept: application/json" -o init_env.sh
                    curl -L -u $NEXUS_USER:$NEXUS_PASSWORD -X GET "$NEXUS_URL:8081/repository/$DEVOPS_SCRIPTS_REPO/conf_nexus_repo.xml" -H "accept: application/json" -o conf_nexus_repo.xml
                    echo curl -L -u $NEXUS_USER:$NEXUS_PASSWORD -X GET "$NEXUS_URL:8081/repository/$DEVOPS_SCRIPTS_REPO/make_params.py" -H "accept: application/json" -o make_params.py
                    curl -L -u $NEXUS_USER:$NEXUS_PASSWORD -X GET "$NEXUS_URL:8081/repository/$DEVOPS_SCRIPTS_REPO/make_params.py" -H "accept: application/json" -o make_params.py
                    
                    chmod +x init_env.sh
                    ls
                    
                    cat init_env.sh
                    ./init_env.sh
                    printenv
                    """
                }
            }
        }

        stage('maven package') {
            steps {
                script{
                    sh '''sudo cat conf_nexus_repo.xml > /opt/maven/conf/settings.xml
                    mvn clean
                    mvn package -DskipTests
                    echo http://${NEXUS_URL}:8081/repository/$DATABASE_URL_PROD/init_env.sh
                    '''
                }
            }
        }
        stage('Hello') {
            steps{
                script{
                    pom = readMavenPom file: "pom.xml";
                    output= mul()
                    echo "The mul is ${output}"
                    sh """ 
                    sudo cat /opt/maven/conf/settings.xml
                    
                    echo $NEXUS_URL:$NEXUS_PORT/repository/$NEXUS_REPOSITORY_NAME/

                    echo "The mul is ${output}"
                    mvn deploy:deploy-file \
                    -Durl=$NEXUS_URL:$NEXUS_PORT/repository/$NEXUS_REPOSITORY_NAME/ \
                    -DrepositoryId=$NEXUS_REPOSITORY_NAME \
                    -DgroupId=${pom.groupId} \
                    -DartifactId=${pom.artifactId} \
                    -Dversion=${pom.version}  \
                    -Dpackaging=jar \
                    -Dfile=target/${pom.name}-${pom.version}.jar
                  """
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

