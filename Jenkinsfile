pipeline {
    agent any

    tools {
        jdk 'JDK_HOME'
        maven 'MAVEN_HOME'
        nodejs 'NODE_18' // Make sure this matches your Jenkins global tools name
    }

    environment {
        BACKEND_DIR = 'ecommerce-backend'
        FRONTEND_DIR = 'ecommerce-frontend'

        TOMCAT_URL = 'http://localhost:9090/manager/text'
        TOMCAT_USER = 'admin'
        TOMCAT_PASS = 'admin'

        BACKEND_WAR = 'springapp2.war'
        FRONTEND_WAR = 'frontapp2.war'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/srithars/ecommerce-fullstack.git', branch: 'master'
            }
        }

        stage('Build Backend (Spring Boot WAR)') {
            steps {
                dir("${env.BACKEND_DIR}") {
                    sh 'mvn clean package'
                    sh "cp target/*.war ../${BACKEND_WAR}"
                }
            }
        }

        stage('Build Frontend (Vite)') {
            steps {
                dir("${env.FRONTEND_DIR}") {
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }

        stage('Package Frontend as WAR') {
            steps {
                dir("${env.FRONTEND_DIR}") {
                    sh """
                        mkdir -p frontapp2_war/WEB-INF
                        cp -r dist/* frontapp2_war/
                        jar -cvf ../../${FRONTEND_WAR} -C frontapp2_war .
                    """
                }
            }
        }

        stage('Undeploy Old Apps from Tomcat') {
            steps {
                sh """
                    curl -u ${TOMCAT_USER}:${TOMCAT_PASS} "${TOMCAT_URL}/undeploy?path=/springapp2" || true
                    curl -u ${TOMCAT_USER}:${TOMCAT_PASS} "${TOMCAT_URL}/undeploy?path=/frontapp2" || true
                """
            }
        }

        stage('Verify WAR Content') {
            steps {
                sh "ls -lh ${BACKEND_WAR}"
                sh "ls -lh ${FRONTEND_WAR}"
            }
        }

        stage('Deploy Backend to Tomcat (/springapp2)') {
            steps {
                sh """
                    curl -u ${TOMCAT_USER}:${TOMCAT_PASS} \\
                         --upload-file ${BACKEND_WAR} \\
                         "${TOMCAT_URL}/deploy?path=/springapp2&update=true"
                """
            }
        }

        stage('Deploy Frontend to Tomcat (/frontapp2)') {
            steps {
                sh """
                    curl -u ${TOMCAT_USER}:${TOMCAT_PASS} \\
                         --upload-file ${FRONTEND_WAR} \\
                         "${TOMCAT_URL}/deploy?path=/frontapp2&update=true"
                """
            }
        }
    }

    post {
        success {
            echo "✅ Backend deployed: http://54.172.97.72:9090/springapp2"
            echo "✅ Frontend deployed: http://54.172.97.72:9090/frontapp2"
        }
        failure {
            echo "❌ Build or deployment failed. Check logs above."
        }
    }
}
