pipeline {
    agent any

    tools {
        jdk 'JDK_HOME'
        maven 'MAVEN_HOME'
    }

    environment {
        BACKEND_DIR = 'crud_backend/crud_backend-main'
        FRONTEND_DIR = 'crud_frontend/crud_frontend-main'
        EC2_HOST = '54.172.97.72'
        TOMCAT_USER = 'admin'
        TOMCAT_PASS = 'admin'
        TOMCAT_PORT = '9090'
        BACKEND_WAR = 'springapp2.war'
        FRONTEND_WAR = 'frontapp2.war'
    }

    stages {
        stage('Build Backend') {
            steps {
                dir("${BACKEND_DIR}") {
                    sh 'mvn clean package'
                    sh "cp target/*.war ../../${BACKEND_WAR}"
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir("${FRONTEND_DIR}") {
                    sh 'mvn clean package'
                    sh "cp target/*.war ../../${FRONTEND_WAR}"
                }
            }
        }

        stage('Undeploy Old Apps from Tomcat') {
            steps {
                script {
                    echo "🧹 Undeploying old applications from Tomcat..."
                    sh "curl -v -u ${TOMCAT_USER}:${TOMCAT_PASS} http://${EC2_HOST}:${TOMCAT_PORT}/manager/text/undeploy?path=/springapp2 || true"
                    sh "curl -v -u ${TOMCAT_USER}:${TOMCAT_PASS} http://${EC2_HOST}:${TOMCAT_PORT}/manager/text/undeploy?path=/frontapp2 || true"
                }
            }
        }

        stage('Verify WAR Content') {
            steps {
                echo "📦 Backend WAR contents:"
                sh "jar -tf ${BACKEND_WAR} | head -n 10 || true"

                echo "📦 Frontend WAR contents:"
                sh "jar -tf ${FRONTEND_WAR} | head -n 10 || true"
            }
        }

        stage('Deploy Backend to Tomcat (/springapp2)') {
            steps {
                echo "🚀 Deploying backend WAR to EC2 Tomcat..."
                sh "curl -v -u ${TOMCAT_USER}:${TOMCAT_PASS} --upload-file ${BACKEND_WAR} http://${EC2_HOST}:${TOMCAT_PORT}/manager/text/deploy?path=/springapp2&update=true"
            }
        }

        stage('Deploy Frontend to Tomcat (/frontapp2)') {
            steps {
                echo "🚀 Deploying frontend WAR to EC2 Tomcat..."
                sh "curl -v -u ${TOMCAT_USER}:${TOMCAT_PASS} --upload-file ${FRONTEND_WAR} http://${EC2_HOST}:${TOMCAT_PORT}/manager/text/deploy?path=/frontapp2&update=true"
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
