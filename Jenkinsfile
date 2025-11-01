node {
 
    def MAVEN_HOME = tool name: 'Maven3'
    def JAVA_HOME = tool name: 'Java21'
    env.PATH = "${MAVEN_HOME}/bin:${JAVA_HOME}/bin:${env.PATH}"

    
    def IMAGE_NAME = "java-app-scripted"
    def DOCKER_HUB_USER = "ahmedhany28"

    try {
        stage('Check Build Number') {
            echo "Current Build Number: ${env.BUILD_NUMBER}"
            if (env.BUILD_NUMBER.toInteger() < 5) {
                error("Build number is less than 5. Failing intentionally!")
            }
        }

        stage('Build with Maven') {
            echo 'Building the Java application...'
            sh 'mvn clean package -DskipTests'
        }

        stage('Docker Build') {
            echo 'Building Docker image...'
            sh "docker build -t ${IMAGE_NAME}:${env.BUILD_NUMBER} ."
        }

        stage('Docker Push') {
            if (env.BUILD_NUMBER.toInteger() >= 5) {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "Logging into Docker Hub..."
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        echo "Tagging image..."
                        docker tag java-app:${BUILD_NUMBER} $DOCKER_USER/java-app:${BUILD_NUMBER}
                        echo "Pushing image..."
                        docker push $DOCKER_USER/java-app:${BUILD_NUMBER}
                        docker logout
                    '''
                }
            } else {
                echo "Skipping Docker push because build number is less than 5."
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
       
        echo "Build FAILED."
        error err.getMessage()
    } finally {
       
        echo 'Cleaning workspace...'
        cleanWs()
    }
}
