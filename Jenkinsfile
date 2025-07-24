pipeline {
    agent any

    tools {
        jdk 'JDK_HOME'
        maven 'MAVEN_HOME'
        nodejs 'NODE_HOME'   // Make sure this matches the name you configured in Jenkins
    }

    environment {
        BACKEND_DIR = 'ecommerce-backend'
        FRONTEND_DIR = 'ecommerce-frontend'
        FRONTEND_WAR = 'frontapp2.war'
        BACKEND_WAR = 'springapp2.war'
        REMOTE_HOST = 'ec2-user@54.172.97.72'
        REMOTE_KEY = '/home/jenkins/fullstack-key.pem'
        REMOTE_TOMCAT = '/opt/tomcat/webapps'
    }

    stages {

        stage('Build Backend (Spring Boot)') {
            steps {
                dir("${BACKEND_DIR}") {
                    sh 'mvn clean package -DskipTests'
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
                        jar -cvf ../${FRONTEND_WAR} -C frontapp2_war .
                    '''
                }
            }
        }

        stage('Verify WAR Content') {
            steps {
                sh 'ls -lh ${BACKEND_WAR}'
                sh 'ls -lh ${FRONTEND_WAR}'
            }
        }

        stage('Deploy WARs to EC2 Tomcat') {
            steps {
                sh '''
                    scp -i ${REMOTE_KEY} ${BACKEND_WAR} ${REMOTE_HOST}:${REMOTE_TOMCAT}/springapp2.war
                    scp -i ${REMOTE_KEY} ${FRONTEND_WAR} ${REMOTE_HOST}:${REMOTE_TOMCAT}/frontapp2.war
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Deployment successful!'
        }
        failure {
            echo '❌ Build or deployment failed. Check logs above.'
        }
    }
}
