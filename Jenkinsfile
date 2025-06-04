pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "duswntmd/tn:1.0"
        GITHUB_REPO = "https://github.com/duswntmd/tn.git"
        JAR_FILE = "tn.jar"
        RELEASE_URL = "https://github.com/duswntmd/tn/releases/download/v1.0.0/tn.jar"
        HOST_UPLOAD_DIR = "/home/ubuntu/uploads"
        CONTAINER_UPLOAD_DIR = "/app/uploads"
    }

    stages {

        stage('Cleanup Workspace') {
            steps {
                cleanWs()
                echo 'Workspace cleaned.'
            }
        }

        stage('Git Clone') {
            steps {
                echo 'Cloning Git repository...'
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: 'main']],
                    userRemoteConfigs: [[
                        credentialsId: 'GitHub_login',
                        url: GITHUB_REPO
                    ]]
                ])
            }
        }

        stage('Download JAR File') {
            steps {
                echo 'Downloading JAR file from GitHub Release...'
                sh """
                    curl -L -o ${JAR_FILE} ${RELEASE_URL}
                    chmod +x ${JAR_FILE}
                """
            }
        }

        stage('Prepare JAR File') {
            steps {
                script {
                    echo "Checking JAR file..."
                    sh """
                        if [ ! -f ${JAR_FILE} ]; then
                            echo 'JAR file not found. Exiting.'
                            exit 1
                        fi
                    """
                }
            }
        }

        stage('Verify JAR File') {
            steps {
                sh "ls -lh ${JAR_FILE}"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Writing Dockerfile and building image..."
                    writeFile file: 'Dockerfile', text: """
                        FROM openjdk:21-slim
                        COPY ${JAR_FILE} /tn.jar
                        EXPOSE 8080
                        VOLUME ${CONTAINER_UPLOAD_DIR}
                        CMD [\"java\", \"-jar\", \"/tn.jar\"]
                    """
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    echo "Stopping old container and running new one..."
                    sh """
                        docker ps -q --filter 'ancestor=${DOCKER_IMAGE}' | xargs --no-run-if-empty docker stop
                        docker run -d \
                            -p 8080:8080 \
                            -v ${HOST_UPLOAD_DIR}:${CONTAINER_UPLOAD_DIR} \
                            --name tn_container \
                            ${DOCKER_IMAGE}
                    """
                }
            }
        }
    }
}
