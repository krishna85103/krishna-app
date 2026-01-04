pipeline {
    agent {
        docker {
            image 'maven:3.9.0-openjdk-17'
            args '-v /root/.m2:/root/.m2'
        }
    }

    environment {
        TOMCAT_LABEL = "app=tomcat-app"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: '<YOUR_GIT_REPO_URL>'
            }
        }

        stage('Build WAR') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    def tomcatPod = sh(
                        script: "kubectl get pods -l ${TOMCAT_LABEL} -o jsonpath='{.items[0].metadata.name}'",
                        returnStdout: true
                    ).trim()

                    sh "kubectl cp target/krishna-app.war ${tomcatPod}:/usr/local/tomcat/webapps/krishna-app.war"
                    sh "kubectl exec ${tomcatPod} -- /usr/local/tomcat/bin/shutdown.sh || true"
                    sh "kubectl exec ${tomcatPod} -- /usr/local/tomcat/bin/startup.sh"
                }
            }
        }
    }

    post {
        success {
            echo "✅ Krishna app deployed successfully!"
        }
        failure {
            echo "❌ Deployment failed."
        }
    }
}

