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

    def JSON_PARAMS="INIT"
    /*registry = '076892551558.dkr.ecr.us-east-1.amazonaws.com/jenkins'
    registryCredential = 'jenkins-ecr'
    dockerimage = ''

     NEXUS_VERSION = "nexus3"
     NEXUS_PROTOCOL = "http"
     NEXUS_URL = "http://ec2-18-237-195-147.us-west-2.compute.amazonaws.com"
     NEXUS_REPOSITORY = "utrains-nexus-pipeline"
     NEXUS_CREDENTIAL_ID = "nexus-user-credentials"
     POM_VERSION = ''*/
}


    stages {
         stage('conf ENV and make') {
            steps {
                script{
                    echo "$NEXUS_URL:8081/repository/$DEVOPS_SCRIPTS_REPO/init_env.sh"
                    sh "sudo /home/ec2-user/ebanking_backend/init_env.sh"
                    echo "test: $NEXUS_USER"
                    
                    sh """#!/bin/bash
                    echo START =======> install_and_config_python_modules
                    sudo yum update -y
                    sudo yum install jq -y
                    sudo yum install python3 pip3 -y
                    sudo pip3 install virtualenv -y
                    sudo /usr/local/bin/python3.11 -m venv ebank
                    source ebank/bin/activate
                    
                    export PYTHONPATH=.
                    sudo pip3 install paramiko
                    sudo pip3 install dog-artifactory --upgrade
                    sudo pip3 install certifi

                    echo START =======> Download scripts to Nexus
                    echo curl -L -u $NEXUS_USER:$NEXUS_PASSWORD -X GET "$NEXUS_URL:8081/repository/$DEVOPS_SCRIPTS_REPO/init_env.sh" -H "accept: application/json" -o init_env.sh
                    echo curl -L -u $NEXUS_USER:$NEXUS_PASSWORD -X GET "$NEXUS_URL:8081/repository/$DEVOPS_SCRIPTS_REPO/conf_nexus_repo.xml" -H "accept: application/json" -o conf_nexus_repo.xml
                    
                    curl -L -u $NEXUS_USER:$NEXUS_PASSWORD -X GET "$NEXUS_URL:8081/repository/$DEVOPS_SCRIPTS_REPO/init_env.sh" -H "accept: application/json" -o init_env.sh
                    curl -L -u $NEXUS_USER:$NEXUS_PASSWORD -X GET "$NEXUS_URL:8081/repository/$DEVOPS_SCRIPTS_REPO/conf_nexus_repo.xml" -H "accept: application/json" -o conf_nexus_repo.xml
                    echo curl -L -u $NEXUS_USER:$NEXUS_PASSWORD -X GET "$NEXUS_URL:8081/repository/$DEVOPS_SCRIPTS_REPO/make_params.py" -H "accept: application/json" -o make_params.py
                    curl -L -u $NEXUS_USER:$NEXUS_PASSWORD -X GET "$NEXUS_URL:8081/repository/$DEVOPS_SCRIPTS_REPO/make_params.py" -H "accept: application/json" -o make_params.py
                    ls
                    
                    echo START =======> Make scripts executable
                    chmod +x init_env.sh
                    cat init_env.sh

                    echo START =======> execute scripts
                    python3 make_params.py pom.xml dev
                    ./init_env.sh
                    ls

                    echo START ===============> Configure ENV Params : 
                    """

                    JSON_PARAMS = readJSON file: 'data.json'

                   sh """

                    echo " START ========================> Configure Nexus xml Credential (REPO ) for the correct environment"
                    sed -i 's/ENV_ROPO_NAME/${JSON_PARAMS.NEXUS_REPO_NAME}/g' conf_nexus_repo.xml
                    sed -i 's/ENV_REPO_USER_NAME/${JSON_PARAMS.NEXUS_USER}/g' conf_nexus_repo.xml
                    sed -i 's/ENV_REPO_PASSWORD/${JSON_PARAMS.NEXUS_PASSWORD}/g' conf_nexus_repo.xml
                    """            
                } 
            }
        }

        stage('maven package') {
            steps {
                script{
                    echo "========================> main test"
                    echo "${JSON_PARAMS.NEXUS_REPO_NAME}"
                    sh '''sudo cat conf_nexus_repo.xml > /opt/maven/conf/settings.xml
                     echo ${APP_VERSION}
                    mvn clean
                    mvn package -DskipTests
                    echo ${NEXUS_URL}:8081/repository/$DATABASE_URL_PROD/init_env.sh
                    '''
                }
            }
        }

        stage('generate compile zip file'){
            steps{
                script{
                    pom = readMavenPom file: "pom.xml";
                    sh """
                        echo sudo zip ${pom.name}-${pom.version}.zip target/ebanking-backend-0.0.1-SNAPSHOT.jar
                        sudo zip ${pom.name}-${pom.version}.zip target/ebanking-backend-0.0.1-SNAPSHOT.jar
                        ls
                        
                    """                  
                }
            }
        }

        stage('Deploy zip file to Nexus'){
            steps{
                script{
                    pom = readMavenPom file: "pom.xml";
                    sh """
                        echo curl -v -u admin:devops --upload-file ${pom.name}-${pom.version}.zip ${NEXUS_URL}:8081/repository/ebanking_dev_zip/${pom.name}/${pom.version}/${pom.name}-${pom.version}.zip
                        curl -v -u admin:devops --upload-file ${pom.name}-${pom.version}.zip ${NEXUS_URL}:8081/repository/ebanking_dev_zip/${pom.name}/${pom.version}/${pom.name}-${pom.version}.zip
                    """                  
                }
            }
        }

        stage('deploy jar to nexus') {
            steps{
                script{
                    pom = readMavenPom file: "pom.xml";
                    output= mul()
                    echo "The mul is ${output}"
                   
                    sh """ 
                    sudo cat /opt/maven/conf/settings.xml
                    
                    echo $NEXUS_URL:$NEXUS_PORT/repository/${JSON_PARAMS.NEXUS_REPO_NAME}/
                    sudo zip test.zip target/ebanking-backend-0.0.1-SNAPSHOT.jar

                    echo "The mul is ${output}"
                    mvn deploy:deploy-file \
                    -Durl=$NEXUS_URL:$NEXUS_PORT/repository/${JSON_PARAMS.NEXUS_REPO_NAME}/ \
                    -DrepositoryId=${JSON_PARAMS.NEXUS_REPO_NAME} \
                    -DgroupId=${pom.groupId} \
                    -DartifactId=${pom.artifactId} \
                    -Dversion=${pom.version}  \
                    -Dpackaging=jar \
                    -DuniqueVersion=false \
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
/*
 {'NEXUS_REPO_NAME': 'ebanking_dev ', 'DEPLOY_HOST_NAME': 'http://url_host_dev ', 'DATABASE_URL': 'jdbc:mysql://localhost ',
  'DATABASE_NAME': 'ebanking_dev', 'DATABASE_PORT': '3306 ', 'OPTION_CREATE_DB_IF_NOT_EXIST': '?createDatabaseIfNotExist=true', 
  'DB_PASSWORD': 'root', 'DB_PORT': '3306', 'DB_USER': 'root', 'NEXUS_USER': 'admin', 'NEXUS_PASSWORD': 'devops', 'APP_VERSION': '0.0.1-SNAPSHOT', 
  'DEVOPS_SCRIPTS_REPO': 'custom_scripts/devops_utils'}*/