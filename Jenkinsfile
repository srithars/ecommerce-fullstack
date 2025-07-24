pipeline {
    agent any

    tools {
        jdk 'JDK_HOME'
        maven 'MAVEN_HOME'
    }

    environment {
        BACKEND_DIR = 'ecommerce-backend'
        FRONTEND_DIR = 'ecommerce-frontend'

        TOMCAT_URL = 'http://54.172.97.72:9090/manager/text'  // Remote EC2 Tomcat
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
                        mkdir -p frontapp2_war/WEB-INF
                        cp -r dist/* frontapp2_war/
                        jar -cvf ../../${FRONTEND_WAR} -C frontapp2_war .
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
                    echo "🧹 Undeploying old applications from Tomcat..."
                    sh """
                        curl -v -u ${TOMCAT_USER}:${TOMCAT_PASS} "${TOMCAT_URL}/undeploy?path=/springapp2"
                        curl -v -u ${TOMCAT_USER}:${TOMCAT_PASS} "${TOMCAT_URL}/undeploy?path=/frontapp2"
                    """
                }
            }
        }

        stage('Verify WAR Content') {
            steps {
                echo "📦 Backend WAR contents:"
                sh "jar -tf ${BACKEND_WAR} | head -n 10"

                echo "📦 Frontend WAR contents:"
                sh "jar -tf ${FRONTEND_WAR} | head -n 10"
            }
        }

        stage('Deploy Backend to Tomcat (/springapp2)') {
            steps {
                script {
                    echo "🚀 Deploying backend WAR to EC2 Tomcat..."
                    sh """
                        curl -v -u ${TOMCAT_USER}:${TOMCAT_PASS} \\
                          --upload-file ${BACKEND_WAR} \\
                          "${TOMCAT_URL}/deploy?path=/springapp2&update=true"
                    """
                }
            }
        }

        stage('Deploy Frontend to Tomcat (/frontapp2)') {
            steps {
                script {
                    echo "🚀 Deploying frontend WAR to EC2 Tomcat..."
                    sh """
                        curl -v -u ${TOMCAT_USER}:${TOMCAT_PASS} \\
                          --upload-file ${FRONTEND_WAR} \\
                          "${TOMCAT_URL}/deploy?path=/frontapp2&update=true"
                    """
                }
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
