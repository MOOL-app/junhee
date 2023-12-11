pipeline {
    agent any

    stages {
        // 새로 추가한 stage
        stage('Permissions') {
            steps {
                // Gradlew 파일에 실행 권한을 부여하는 단계
                sh 'chmod +x ./gradlew'
            }
        }

        stage('Build Spring Boot Project') {
            steps {
                script {
                    // Spring Boot 프로젝트 빌드
                    sh './gradlew clean build --warning-mode all'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Docker 이미지 빌드
                    docker.build("joiejuni/test", ".")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Docker 이미지를 Docker Hub로 푸시
                    docker.withRegistry('https://hub.docker.com/repository/docker', 'joiejuni') {
                        docker.image("joiejuni/test").push()
                    }
                }
            }
        }

        stage('Stop and Remove Existing Container') {
            steps {
                script {
                    // 기존에 동작 중인 컨테이너 중지 및 삭제
                    sh 'docker ps -q --filter "name=spring-boot-server" | grep -q . && docker stop spring-boot-server && docker rm spring-boot-server || true'
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    // Docker 컨테이너 실행
                    sh 'docker run -p 8081:8080 -d --name=spring-boot-server joiejuni/test'
                }
            }
        }

        stage('Clean Up Unused Docker Images') {
            steps {
                script {
                    // 태그가 겹친 이미지 삭제
                    sh 'docker rmi -f $(docker images -f "dangling=true" -q) || true'
                }
            }
        }
    }
}