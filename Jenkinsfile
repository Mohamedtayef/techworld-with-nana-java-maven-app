@library('jenkins-shared')_
pipeline {
    agent any
    tools {
        maven: "Maven"
    }
    stages {

        stage("increment version") {
            steps {
                script {
                    echo 'incrementing app version...'
                    sh 'mvn build-helper:parse-version versions:set -DnewVersion=\${parsedVersion.majorVersion}.\${parsedVersion.minorVersion}.\${parsedVersion.nextIncrementalVersion} versions:commit'
                    def matcher = readFile('pom.xml') =~ /<version>(.*)<\/version>/
                    def version = matcher[0][1]
                    env.IMAGE_NAME = env.IMAGE_NAME + "$version-$buildNumber"
                }
            }
        }
        stage("build jar") {
            steps {
                script {
                    echo "building the application..."
                    buildJar()
                }
            }
        }
        stage("build image") {
            steps {
                script {
                    echo "building the docker image..."
                    buildImage(env.IMAGE_NAME)
                    dockerLogin()
                    dockerPush(env.IMAGE_NAME)
            }
        }
        stage("provision server") {
            environment {
                AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id')
                AWS_SECRET_ACCESS_KEY = credentials('jenkins_aws_secret_access_key')
                APP_NAME = "java-maven-app"

            }
            steps {
                script {
                    dir('terraform'){
                        sh "terraform init"        
                        sh "terraform apply --auto-approve"
                        EC2_PUBLIC_IP = sh(
                         script: "terraform output aws_public_ip"
                         returnStdout: true   
                        ).trim()

                    }
            }
        }
        stage("deploy") {
            environment {
                DOCKER_CREDS =credentials('docker-hub-cred')
            }
            steps {
                script {
                    sleep(time: 90, unit: "SECONDS")
                    def shellCmd = "bash ./server.shexport ${IMAGE_NAME} ${DOCKER_CREDS_USER} ${DOCKER_CREDS_PWD}"
                    def ec2Instance = "ec2-user@${EC2_PUBLIC_Ip}"
                    sshagent(['ec2-server-key'])
                    echo "Deploy docker image..."
                    sh "envsubst < kubernetes/deployment.yaml kubectl apply -f -"
                    sh "envsubst < kubernetes/service.yaml kubectl apply -f -"
                }
            }
        }
    }   
}
