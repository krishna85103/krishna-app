pipeline {
    agent any

    tools {
        maven 'maven 3'
    }

    environment {
        GIT_REPO     = 'https://github.com/krishna85103/krishna-app.git'
        GIT_BRANCH  = 'main'
        TOMCAT_LABEL = 'app=tomcat-app'
        WAR_NAME    = 'krishna-app.war'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: "${GIT_BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Build WAR with Maven') {
            steps {
                echo "Building WAR using Jenkins-managed Maven..."
                sh '''
                  mvn -version
                  mvn clean package
                '''
            }
        }

        stage('Deploy to Tomcat (Kubernetes)') {
            steps {
                script {
                    def tomcatPod = sh(
                        script: "kubectl get pods -l ${TOMCAT_LABEL} -o jsonpath='{.items[0].metadata.name}'",
                        returnStdout: true
                    ).trim()

                    echo "Tomcat Pod: ${tomcatPod}"

                    sh """
                      kubectl cp target/${WAR_NAME} ${tomcatPod}:/usr/local/tomcat/webapps/${WAR_NAME}
                      kubectl exec ${tomcatPod} -- /usr/local/tomcat/bin/shutdown.sh || true
                      kubectl exec ${tomcatPod} -- /usr/local/tomcat/bin/startup.sh
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Krishna app built & deployed successfully!"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}

