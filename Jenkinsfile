pipeline {
    agent any
    environment {
        PROJECT_ID = 'opensource-398710'
        CLUSTER_NAME = 'kube'
        LOCATION = 'asia-northeast3-a'
        CREDENTIALS_ID = '5c0d9183-3035-46ac-a72b-a4d4b373f84d'
    }
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
                    docker.build("joiejuni/mool", ".")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Docker 이미지를 Docker Hub로 푸시
                    docker.withRegistry('https://registry.hub.docker.com', 'joiejuni') {
                        docker.image("joiejuni/mool").push()
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
                    sh 'docker run -p 8081:8080 -d --name=spring-boot-server joiejuni/mool:${env.BUILD_ID}'
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

        stage('Deploy to GKE') {
			// when {
			// 	branch 'main'
			// }
            steps{
                sh "sed -i 's/mool:latest/mool:${env.BUILD_ID}/g' deployment.yaml"
                step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'deployment.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
            }
        }

    }
}
