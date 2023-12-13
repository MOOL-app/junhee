pipeline {
    agent any
    environment {
        PROJECT_ID = 'opensource-398710'
        CLUSTER_NAME = 'kube'
        LOCATION = 'asia-northeast3-a'
        CREDENTIALS_ID = '5c0d9183-3035-46ac-a72b-a4d4b373f84d'
    }
    stages {

        stage('Build Docker Image') {
            steps {
                script {
                    // Docker 이미지 빌드
                    myapp=docker.build("joiejuni/mool:${env.BUILD_ID}", ".")
                }
            }
        }


        stage('Push Docker Image') {
            steps {
                script {
                    // Docker 이미지를 Docker Hub로 푸시
                    docker.withRegistry('https://registry.hub.docker.com', 'joiejuni') {
			    myapp.push("latest")
			    myapp.push("${env.BUILD_ID}")
                        //docker.image("joiejuni/mool").push()
                    }
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
