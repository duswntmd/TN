pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "duswntmd/tn:1.0"
        GITHUB_REPO = "https://github.com/duswntmd/tn.git"
        JAR_FILE = "tn.jar"
        RELEASE_URL = "https://github.com/duswntmd/tn/releases/download/v1.0.0/tn.jar"
    }

    stages {

        stage('Cleanup Workspace') {
            steps {
                echo 'Cleaned up'
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
                    echo "Preparing JAR file..."
                    sh """
                    if [ -f ${JAR_FILE} ]; then
                        echo "JAR file ${JAR_FILE} found."
                    else
                        echo "JAR file not found. Ensure the Release URL is correct."
                        exit 1
                    fi
                    """
                }
            }
        }

        stage('Verify JAR File') {
            steps {
                echo "Verifying JAR file..."
                sh "ls -l ${JAR_FILE}"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image..." 
                    writeFile file: 'Dockerfile', text: """
                    FROM openjdk:21-slim
                    COPY ${JAR_FILE} /tn.jar
                    EXPOSE 80
                    CMD ["java", "-jar", "/tn.jar"]
                    """
                    sh "docker build -t ${DOCKER_IMAGE} ."
                    echo "Docker image created!"
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    echo "Running Docker container on port 80..."
                    sh """
                    docker ps -q --filter 'ancestor=${DOCKER_IMAGE}' | xargs --no-run-if-empty docker stop
                    docker run -d -p 8080:8080 ${DOCKER_IMAGE}
                    """
                }
            }
        }
    }
}
