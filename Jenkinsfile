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
         stage('MAKE ENV') {
            steps {
                script{
                    echo "$NEXUS_URL:8081/repository/$DEVOPS_SCRIPTS_REPO/init_env.sh"
                    sh "sudo /home/ec2-user/ebanking_backend/init_env.sh"
                    echo "test: $NEXUS_USER"
                    
                    sh """#!/bin/bash
                    echo START =======> install_and_config_python_modules
                    sudo yum update -y
                    sudo yum install jq -y
                    python3 -m venv ebank
                    source ebank/bin/activate
                    
                    export PYTHONPATH=.
                    pip3 install paramiko
                    pip3 install certifi

                    echo START =======> Download scripts to Nexus
                    echo curl -L -u $NEXUS_USER:$NEXUS_PASSWORD -X GET "$NEXUS_URL:8081/repository/$DEVOPS_SCRIPTS_REPO/init_env.sh" -H "accept: application/json" -o init_env.sh
                    echo curl -L -u $NEXUS_USER:$NEXUS_PASSWORD -X GET "$NEXUS_URL:8081/repository/$DEVOPS_SCRIPTS_REPO/conf_nexus_repo.xml" -H "accept: application/json" -o conf_nexus_repo.xml
                    
                    curl -L -u $NEXUS_USER:$NEXUS_PASSWORD -X GET "$NEXUS_URL:8081/repository/$DEVOPS_SCRIPTS_REPO/init_env.sh" -H "accept: application/json" -o init_env.sh
                    curl -L -u $NEXUS_USER:$NEXUS_PASSWORD -X GET "$NEXUS_URL:8081/repository/$DEVOPS_SCRIPTS_REPO/conf_nexus_repo.xml" -H "accept: application/json" -o conf_nexus_repo.xml
                    echo curl -L -u $NEXUS_USER:$NEXUS_PASSWORD -X GET "$NEXUS_URL:8081/repository/$DEVOPS_SCRIPTS_REPO/make_params.py" -H "accept: application/json" -o make_params.py
                    curl -L -u $NEXUS_USER:$NEXUS_PASSWORD -X GET "$NEXUS_URL:8081/repository/$DEVOPS_SCRIPTS_REPO/make_params.py" -H "accept: application/json" -o make_params.py
                    
                    curl -L -u $NEXUS_USER:$NEXUS_PASSWORD -X GET "$NEXUS_URL:8081/repository/$DEVOPS_SCRIPTS_REPO/stop_ebank.sh" -H "accept: application/json" -o stop_ebank.sh
                    curl -L -u $NEXUS_USER:$NEXUS_PASSWORD -X GET "$NEXUS_URL:8081/repository/$DEVOPS_SCRIPTS_REPO/start_ebank.sh" -H "accept: application/json" -o start_ebank.sh
                    curl -L -u $NEXUS_USER:$NEXUS_PASSWORD -X GET "$NEXUS_URL:8081/repository/$DEVOPS_SCRIPTS_REPO/start_app.py" -H "accept: application/json" -o start_app.py
                    
                    curl -L -u $NEXUS_USER:$NEXUS_PASSWORD -X GET "$NEXUS_URL:8081/repository/$DEVOPS_SCRIPTS_REPO/upload_file_to_server.py" -H "accept: application/json" -o upload_file_to_server.py

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

        stage('MAVEN PACKAGE') {
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

        stage('MAKE ZIP FILE'){
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

        stage('PUSH ARTIFACTS : NEXUS ZIP FILE'){
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

        stage('PUSH ARTIFACTS : NEXUS') {
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
                    -DuniqueVersion=false \
                    -DpomFile=pom.xml \
                    -Dversion=${pom.version}  \
                    -Dpackaging=jar \
                    -Dfile=target/${pom.name}-${pom.version}.jar
                  """
            }
            }       
        }

        /*
        This course enables you to upload the jar file to the environment where it will be deployed. 
        Then copy the installation scripts from this jar file to the environment in order to prepare the installation.
        */
        stage('DEPLOY APP'){
            steps{
                script{
                    pom = readMavenPom file: "pom.xml";
                    sh """
                        python3 -m venv ebank
                        source ebank/bin/activate
                    
                        export PYTHONPATH=.
                        pip3 install paramiko
                        pip3 install certifi
                    
                        echo ${JSON_PARAMS.DEPLOY_HOST_NAME}
                        echo ${JSON_PARAMS.APP_USER}
                        echo ${JSON_PARAMS.APP_PASSWORD}
                        echo ${JSON_PARAMS.APP_PATH}
                        echo ${pom.name}-${pom.version}.jar
                        printenv
                        echo Pulling... $GIT_BRANCH
                        python3 upload_file_to_server.py ${JSON_PARAMS.DEPLOY_HOST_NAME} ${JSON_PARAMS.APP_USER} ${JSON_PARAMS.APP_PASSWORD} ${JSON_PARAMS.APP_PATH} target/${pom.name}-${pom.version}.jar ${pom.name}-${pom.version}.jar
                        python3 upload_file_to_server.py ${JSON_PARAMS.DEPLOY_HOST_NAME} ${JSON_PARAMS.APP_USER} ${JSON_PARAMS.APP_PASSWORD} ${JSON_PARAMS.APP_PATH} start_ebank.sh start_ebank.sh
                        python3 upload_file_to_server.py ${JSON_PARAMS.DEPLOY_HOST_NAME} ${JSON_PARAMS.APP_USER} ${JSON_PARAMS.APP_PASSWORD} ${JSON_PARAMS.APP_PATH} stop_ebank.sh stop_ebank.sh
                        python3 upload_file_to_server.py ${JSON_PARAMS.DEPLOY_HOST_NAME} ${JSON_PARAMS.APP_USER} ${JSON_PARAMS.APP_PASSWORD} ${JSON_PARAMS.APP_PATH} init_env.sh init_env.sh                          
                    """                  
                }
            }
        }

        stage('LUNCH APP'){
            steps{
                script{
                    pom = readMavenPom file: "pom.xml";
                    sh """
                        python3 -m venv ebank
                        source ebank/bin/activate
                    
                        export PYTHONPATH=.
                        pip3 install paramiko
                        pip3 install certifi

                        echo Pulling... $GIT_BRANCH
                        printenv
                        echo python3 start_app.py ${JSON_PARAMS.DEPLOY_HOST_NAME} ${JSON_PARAMS.APP_USER} ${JSON_PARAMS.APP_PASSWORD} ${JSON_PARAMS.APP_PATH} ${JSON_PARAMS.NEXUS_REPO_NAME} start_ebank.sh ${pom.name}-${pom.version}.jar jar
                        python3 start_app.py ${JSON_PARAMS.DEPLOY_HOST_NAME} ${JSON_PARAMS.APP_USER} ${JSON_PARAMS.APP_PASSWORD} ${JSON_PARAMS.APP_PATH} ${JSON_PARAMS.NEXUS_REPO_NAME} start_ebank.sh ${pom.name}-${pom.version}.jar jar                      
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