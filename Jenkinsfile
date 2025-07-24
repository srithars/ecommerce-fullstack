pipeline {
    agent any

    tools {
        jdk 'JDK_HOME'
        maven 'MAVEN_HOME'
    }

    environment {
        BACKEND_DIR = 'ecommerce-backend'
        FRONTEND_DIR = 'ecommerce-frontend'

        TOMCAT_USER = 'ec2-user'
        TOMCAT_HOST = '54.172.97.72'
        TOMCAT_KEY = '/home/jenkins/fullstack-key.pem'

        BACKEND_WAR = 'target/springapp2.war'
        FRONTEND_WAR = 'dist/frontapp2.war'
    }

    stages {
        stage('Build Backend') {
            steps {
                dir("${BACKEND_DIR}") {
                    sh 'mvn clean package'
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir("${FRONTEND_DIR}") {
                    sh '''
                        npm install
                        npm run build
                    '''
                }
            }
        }

        stage('Undeploy Old Apps from Tomcat') {
            steps {
                sh """
                ssh -i ${TOMCAT_KEY} ${TOMCAT_USER}@${TOMCAT_HOST} 'rm -rf /opt/tomcat/webapps/springapp2*'
                ssh -i ${TOMCAT_KEY} ${TOMCAT_USER}@${TOMCAT_HOST} 'rm -rf /opt/tomcat/webapps/frontapp2*'
                """
            }
        }

        stage('Verify WAR Content') {
            steps {
                sh "ls -l ${BACKEND_DIR}/${BACKEND_WAR}"
                sh "ls -l ${FRONTEND_DIR}/${FRONTEND_WAR}"
            }
        }

        stage('Deploy Backend to Tomcat (/springapp2)') {
            steps {
                sh """
                scp -i ${TOMCAT_KEY} ${BACKEND_DIR}/${BACKEND_WAR} ${TOMCAT_USER}@${TOMCAT_HOST}:/opt/tomcat/webapps/springapp2.war
                """
            }
        }

        stage('Deploy Frontend to Tomcat (/frontapp2)') {
            steps {
                sh """
                cd ${FRONTEND_DIR}
                zip -r frontapp2.war dist
                scp -i ${TOMCAT_KEY} frontapp2.war ${TOMCAT_USER}@${TOMCAT_HOST}:/opt/tomcat/webapps/frontapp2.war
                """
            }
        }
    }

    post {
        failure {
            echo '❌ Build or deployment failed. Check logs above.'
        }
        success {
            echo '✅ Build and deployment successful!'
        }
    }
}
