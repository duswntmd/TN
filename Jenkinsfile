pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "duswntmd/tn:1.0"
        GITHUB_REPO = "https://github.com/duswntmd/tn.git"
        JAR_FILE = "tn.jar"
        RELEASE_URL = "https://github.com/duswntmd/tn/releases/latest/download/tn.jar"
        HOST_UPLOAD_DIR = "/home/ubuntu/uploads"
        CONTAINER_UPLOAD_DIR = "/app/uploads"
        CONTAINER_NAME = "tn_container"
    }

    stages {
        stage('Cleanup Workspace') {
            steps {
                cleanWs()
                echo '✅ Jenkins 워크스페이스 정리 완료'
            }
        }

        stage('Git Clone') {
            steps {
                echo '📦 Git 저장소 클론 중...'
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
                echo '⬇️ GitHub Release에서 JAR 파일 다운로드 중...'
                sh """
                    curl -L -o ${JAR_FILE} ${RELEASE_URL}
                    chmod +x ${JAR_FILE}
                """
            }
        }

        stage('Verify JAR File') {
            steps {
                echo "🔍 JAR 파일 존재 여부 확인"
                sh """
                    if [ ! -f ${JAR_FILE} ]; then
                        echo '❌ JAR 파일이 없습니다. 종료합니다.'
                        exit 1
                    fi
                    ls -lh ${JAR_FILE}
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "🐳 Dockerfile 생성 및 이미지 빌드 중..."
                    writeFile file: 'Dockerfile', text: """
                        FROM openjdk:21-slim
                        COPY ${JAR_FILE} /tn.jar
                        VOLUME ${CONTAINER_UPLOAD_DIR}
                        EXPOSE 8080
                        CMD ["java", "-jar", "/tn.jar"]
                    """
                    sh "docker build --no-cache -t ${DOCKER_IMAGE} ."
                    echo "✅ Docker 이미지 빌드 완료"
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    echo "🚀 기존 컨테이너 정리 후 새 컨테이너 실행"

                    sh """
                        # 기존 컨테이너 중지 및 제거
                        docker stop ${CONTAINER_NAME} || true
                        docker rm ${CONTAINER_NAME} || true

                        # 8080 포트 충돌 방지 - 해당 포트 사용하는 모든 컨테이너 중지
                        docker ps --filter "publish=8080" -q | xargs -r docker stop || true

                        # 새 컨테이너 실행
                        docker run -d \
                            -p 8080:8080 \
                            -v ${HOST_UPLOAD_DIR}:${CONTAINER_UPLOAD_DIR} \
                            --name ${CONTAINER_NAME} \
                            ${DOCKER_IMAGE}
                    """
                    echo "✅ Docker 컨테이너 실행 완료"
                }
            }
        }
    }
}
