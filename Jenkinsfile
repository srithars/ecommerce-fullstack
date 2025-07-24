pipeline {
    agent any

    tools {
        jdk 'JDK_HOME'
        maven 'MAVEN_HOME'
        nodejs 'NODE_HOME'
    }

    environment {
        BACKEND_DIR = 'ecommerce-backend'
        FRONTEND_DIR = 'ecommerce-frontend'
        BACKEND_WAR = 'springapp2.war'
        FRONTEND_WAR = 'frontapp2.war'
        TOMCAT_USER = 'admin'
        TOMCAT_PASS = 'admin'
        TOMCAT_HOST = 'localhost'
        TOMCAT_PORT = '9090'
        TOMCAT_WEBAPPS_PATH = '/opt/tomcat/webapps'
        EC2_HOST = '54.172.97.72'
        SSH_KEY = '/home/jenkins/fullstack-key.pem'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/srithars/ecommerce-fullstack.git'
            }
        }

        stage('Build Backend (Maven)') {
            steps {
                dir("${BACKEND_DIR}") {
                    sh 'mvn clean package'
                    sh "cp target/*.war ../${BACKEND_WAR}"
                }
            }
        }

        stage('Build Frontend (Vite)') {
            steps {
                dir("${FRONTEND_DIR}") {
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }

        stage('Package Frontend as WAR') {
            steps {
                dir("${FRONTEND_DIR}") {
                    sh '''
                        mkdir -p frontapp2_war/WEB-INF
                        cp -r dist/* frontapp2_war/
                        jar -cvf frontapp2.war -C frontapp2_war .
                        mv frontapp2.war ../../${FRONTEND_WAR}
                    '''
                }
            }
        }

        stage('Undeploy Old Apps from Tomcat') {
            steps {
                sh "curl -u ${TOMCAT_USER}:${TOMCAT_PASS} http://${TOMCAT_HOST}:${TOMCAT_PORT}/manager/text/undeploy?path=/springapp2 || true"
                sh "curl -u ${TOMCAT_USER}:${TOMCAT_PASS} http://${TOMCAT_HOST}:${TOMCAT_PORT}/manager/text/undeploy?path=/frontapp2 || true"
            }
        }

        stage('Verify WAR Content') {
            steps {
                sh 'ls -lh *.war'
            }
        }

        stage('Deploy Backend to Tomcat (/springapp2)') {
            steps {
                sh "scp -i ${SSH_KEY} ${BACKEND_WAR} ec2-user@${EC2_HOST}:${TOMCAT_WEBAPPS_PATH}/springapp2.war"
            }
        }

        stage('Deploy Frontend to Tomcat (/frontapp2)') {
            steps {
                sh "scp -i ${SSH_KEY} ${FRONTEND_WAR} ec2-user@${EC2_HOST}:${TOMCAT_WEBAPPS_PATH}/frontapp2.war"
            }
        }
    }

    post {
        success {
            echo '✅ Build and deployment successful!'
        }
        failure {
            echo '❌ Build or deployment failed. Check logs above.'
        }
    }
}
