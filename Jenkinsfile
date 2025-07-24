pipeline {
    agent any

    tools {
        jdk 'JDK_HOME'
        maven 'MAVEN_HOME'
        nodejs 'NODE_HOME'  // Node.js 18.20.8 configured in Jenkins
    }

    environment {
        BACKEND_DIR = 'ecommerce-backend'
        FRONTEND_DIR = 'ecommerce-frontend'
        BACKEND_WAR = 'springapp2.war'
        FRONTEND_WAR = 'frontapp2.war'
        REMOTE_USER = 'ec2-user'
        REMOTE_HOST = '54.172.97.72'
        REMOTE_PATH = '/opt/tomcat/webapps'
        SSH_KEY = '/home/jenkins/fullstack-key.pem'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/srithars/ecommerce-fullstack.git'
            }
        }

        stage('Build Backend') {
            steps {
                dir("${BACKEND_DIR}") {
                    sh 'mvn clean install'
                    sh "cp target/*.war ../${BACKEND_WAR}"
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir("${FRONTEND_DIR}") {
                    sh 'npm install'
                    sh 'npm run build'
                    sh 'mkdir -p dist/WEB-INF && echo "<web-app/>" > dist/WEB-INF/web.xml'
                    sh "jar -cvf ../../${FRONTEND_WAR} -C dist ."
                }
            }
        }

        stage('Undeploy Old Apps from Tomcat') {
            steps {
                sh """
                    ssh -o StrictHostKeyChecking=no -i ${SSH_KEY} ${REMOTE_USER}@${REMOTE_HOST} 'rm -rf ${REMOTE_PATH}/springapp2* ${REMOTE_PATH}/frontapp2*'
                """
            }
        }

        stage('Verify WAR Files') {
            steps {
                sh 'ls -lh *.war'
            }
        }

        stage('Deploy Backend (/springapp2)') {
            steps {
                sh """
                    scp -o StrictHostKeyChecking=no -i ${SSH_KEY} ${BACKEND_WAR} ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_PATH}/
                """
            }
        }

        stage('Deploy Frontend (/frontapp2)') {
            steps {
                sh """
                    scp -o StrictHostKeyChecking=no -i ${SSH_KEY} ${FRONTEND_WAR} ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_PATH}/
                """
            }
        }
    }

    post {
        success {
            echo '✅ Deployment to Tomcat successful!'
        }
        failure {
            echo '❌ Deployment failed. Please check logs.'
        }
    }
}
