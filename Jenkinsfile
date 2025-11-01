node {
    // === Setup tools ===
    def mvnHome = tool name: 'Maven3', type: 'maven'
    def jdkHome = tool name: 'Java21', type: 'hudson.model.JDK'
    env.PATH = "${jdkHome}/bin:${mvnHome}/bin:${env.PATH}"

    // === Environment variables ===
    env.IMAGE_NAME = "java-app"
    env.DOCKER_HUB_USER = "ahmedhany28"

    try {
        stage('Checkout code') {
            echo "Checking out the public repository..."
            checkout([
                $class: 'GitSCM',
                branches: [[name: '*/main']],
                userRemoteConfigs: [[url: 'https://github.com/A7medHanyFCAI/Scripted-Pipeline.git']]
            ])
            sh 'pwd'
            sh 'ls -la'
        }

        stage('Check Build Number') {
            echo "Current Build Number: ${env.BUILD_NUMBER}"
        }

        stage('Build with Maven') {
            echo 'Building the Java application...'
            sh "'${mvnHome}/bin/mvn' clean package -DskipTests"
        }

        stage('Docker Build & Push') {
            echo 'Building and pushing Docker image...'
            withCredentials([usernamePassword(
                credentialsId: 'dockerhub',
                usernameVariable: 'DOCKER_USER',
                passwordVariable: 'DOCKER_PASS'
            )]) {
                sh """
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                    docker tag ${IMAGE_NAME}:${BUILD_NUMBER} $DOCKER_USER/${IMAGE_NAME}:${BUILD_NUMBER}
                    docker push $DOCKER_USER/${IMAGE_NAME}:${BUILD_NUMBER}
                    docker logout
                """
            }
        }

        stage('Deploy') {
            echo 'Deploying the container...'
            sh """
                docker rm -f java-app || true
                docker run -d --name java-app -p 8090:8090 ${DOCKER_HUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}
            """
        }

        echo '✅ Build SUCCESS!'

    } catch (err) {
        echo "❌ Build FAILED: ${err}"
        currentBuild.result = 'FAILURE'
        throw err
    } finally {
        echo '🧹 Cleaning workspace...'
        cleanWs()
    }
}
