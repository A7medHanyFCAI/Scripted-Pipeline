node {
    def mvnHome = tool name: 'Maven3', type: 'maven'
    def jdkHome = tool name: 'Java21', type: 'hudson.model.JDK'
    env.PATH = "${jdkHome}/bin:${mvnHome}/bin:${env.PATH}"

    env.IMAGE_NAME = "java-app"
    env.DOCKER_HUB_USER = "ahmedhany28"

    try {
        stage('Check Build Number') {
            echo "Current Build Number: ${env.BUILD_NUMBER}"
            if (env.BUILD_NUMBER.toInteger() < 5) {
                error("Build number is less than 5. Failing intentionally!")
            }
        }

        stage('Build with Maven') {
            echo 'Building the Java application...'
            dir('Scripted-Pipeline') {  
                sh "'${mvnHome}/bin/mvn' clean package -DskipTests"
            }
        }

        stage('Docker Build') {
            echo 'Building Docker image...'
            dir('Scripted-Pipeline') {  
                sh "docker build -t ${env.IMAGE_NAME}:${env.BUILD_NUMBER} ."
            }
        }

        stage('Docker Push') {
            if (env.BUILD_NUMBER.toInteger() >= 5) {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    dir('Scripted-Pipeline') { 
                        sh '''
                            echo "Logging into Docker Hub..."
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            echo "Tagging image..."
                            docker tag ${IMAGE_NAME}:${BUILD_NUMBER} $DOCKER_USER/${IMAGE_NAME}:${BUILD_NUMBER}
                            echo "Pushing image to Docker Hub..."
                            docker push $DOCKER_USER/${IMAGE_NAME}:${BUILD_NUMBER}
                            docker logout
                        '''
                    }
                }
            }
        }

        stage('Deploy on Docker') {
            echo 'Deploying the container...'
            sh """
                docker rm -f java-app || true
                docker run -d --name java-app -p 8090:8090 ${DOCKER_HUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}
            """
        }

        echo 'Build SUCCESS!'

    } catch (err) {
        echo "Build FAILED: ${err}"
        currentBuild.result = 'FAILURE'
    } finally {
        echo 'Cleaning workspace...'
        cleanWs()
    }
}
