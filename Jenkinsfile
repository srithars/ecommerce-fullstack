pipeline {
    agent any

    tools {
        jdk 'JDK_HOME'
        maven 'MAVEN_HOME'
    }

    environment {
        BACKEND_DIR = 'ecommerce-backend'
        FRONTEND_DIR = 'ecommerce-frontend'

        TOMCAT_URL = 'http://localhost:9090/manager/text'
        TOMCAT_USER = 'admin'
        TOMCAT_PASS = 'admin'

        BACKEND_WAR = 'springapp1.war'
        FRONTEND_WAR = 'frontapp1.war'

        EC2_USER = 'ec2-user'
        EC2_IP = '54.172.97.72'
        PEM_PATH = '/home/jenkins/fullstack-key.pem'  // 🔁 Replace with correct path on Jenkins server
        TOMCAT_WEBAPPS = '/opt/tomcat/webapps'        // 🔁 Update if your Tomcat path is different
    }

    stages {
        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/srithars/ecommerce-fullstack.git', branch: 'master'
            }
        }

        stage('Build Frontend (Vite)') {
            steps {
                dir("${env.FRONTEND_DIR}") {
                    script {
                        def nodeHome = tool name: 'NODE_HOME', type: 'jenkins.plugins.nodejs.tools.NodeJSInstallation'
                        env.PATH = "${nodeHome}/bin:${env.PATH}"
                    }
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }

        stage('Package Frontend as WAR') {
            steps {
                dir("${env.FRONTEND_DIR}") {
                    sh """
                        mkdir -p frontapp1_war/WEB-INF
                        cp -r dist/* frontapp1_war/
                        jar -cvf ../../${FRONTEND_WAR} -C frontapp1_war .
                    """
                }
            }
        }

        stage('Build Backend (Spring Boot WAR)') {
            steps {
                dir("${env.BACKEND_DIR}") {
                    sh 'mvn clean package'
                    sh "cp target/*.war ../../${BACKEND_WAR}"
                }
            }
        }

        stage('Undeploy Old Apps from Tomcat') {
            steps {
                script {
                    sh """
                        curl -s -u ${TOMCAT_USER}:${TOMCAT_PASS} "${TOMCAT_URL}/undeploy?path=/springapp1"
                        curl -s -u ${TOMCAT_USER}:${TOMCAT_PASS} "${TOMCAT_URL}/undeploy?path=/frontapp1"
                    """
                }
            }
        }

        stage('Clean Tomcat Webapps on EC2') {
            steps {
                script {
                    sh """
                        ssh -i ${PEM_PATH} -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} '
                          sudo rm -rf ${TOMCAT_WEBAPPS}/springapp1.war ${TOMCAT_WEBAPPS}/springapp1
                          sudo rm -rf ${TOMCAT_WEBAPPS}/frontapp1.war ${TOMCAT_WEBAPPS}/frontapp1
                        '
                    """
                }
            }
        }

        stage('Deploy Backend to Tomcat (/springapp1)') {
            steps {
                script {
                    sh """
                        curl -s -u ${TOMCAT_USER}:${TOMCAT_PASS} \\
                          --upload-file ${BACKEND_WAR} \\
                          "${TOMCAT_URL}/deploy?path=/springapp1"
                    """
                }
            }
        }

        stage('Deploy Frontend to Tomcat (/frontapp1)') {
            steps {
                script {
                    sh """
                        curl -s -u ${TOMCAT_USER}:${TOMCAT_PASS} \\
                          --upload-file ${FRONTEND_WAR} \\
                          "${TOMCAT_URL}/deploy?path=/frontapp1"
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Backend deployed: http://54.172.97.72:9090/springapp1"
            echo "✅ Frontend deployed: http://54.172.97.72:9090/frontapp1"
        }
        failure {
            echo "❌ Build or deployment failed"
        }
    }
}
