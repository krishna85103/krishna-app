pipeline {
    agent {
        docker {
            image 'maven:3.9.9-eclipse-temurin-17'
            args '-v /root/.m2:/root/.m2'
        }
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

        stage('Build WAR') {
            steps {
                echo "Building WAR using Maven Docker agent..."
                sh 'mvn clean package'
                sh 'ls -l target'
            }
        }

        stage('Deploy to Tomcat (Kubernetes)') {
            steps {
                script {
                    def tomcatPod = sh(
                        script: "kubectl get pods -l ${TOMCAT_LABEL} -o jsonpath='{.items[0].metadata.name}'",
                        returnStdout: true
                    ).trim()

                    echo "Tomcat Pod detected: ${tomcatPod}"

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
            echo "✅ Krishna app deployed successfully to Tomcat!"
        }
        failure {
            echo "❌ Build or deployment failed. Check logs."
        }
    }
}

