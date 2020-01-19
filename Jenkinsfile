pipeline {
    agent any 

    environment {
        DOCKER_USERNAME = credentials('acr-secret-username')
        DOCKER_PASSWORD = credentials('acr-secret-password')
        DOCKER_REGISTRY = 'mydevopsacr001.azurecr.io'
        PROJECT_NAME = 'example-product-service'
        K8S_NAMESPACE = 'deployment'
        DOCKER_IMAGE = "${DOCKER_REGISTRY}/${PROJECT_NAME}:${env.BUILD_TIMESTAMP}.${env.BUILD_ID}"
        DOCKER_LATEST_IMAGE = "${DOCKER_REGISTRY}/${PROJECT_NAME}:latest"
        JIB_IMAGE = "net.thoughtworks/${PROJECT_NAME}:latest"
    }

    stages {
        stage('clone repository') {
        	sh 'echo "Cloning GitHub repository ..."'
        	checkout scm
        }

        stage('gradle clean build') {
            steps {
                sh './gradlew clean build'
            }
        }
        stage('push image to ACR') {
            steps {
                sh """
                ./gradlew jibDockerBuild
                docker tag ${JIB_IMAGE} ${DOCKER_IMAGE}
                docker tag ${JIB_IMAGE} ${DOCKER_LATEST_IMAGE}
                docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
                docker push ${DOCKER_IMAGE}
                docker push ${DOCKER_LATEST_IMAGE}
                """
            }
        }

        stage('deploy to dev AKS') {
        		sh 'echo "Using Kubectl to upgrade application (redeploy) on AKS"'
        		withKubeConfig(
        			caCertificate: '',
        			contextName: 'dev',
        			credentialsId: 'aks-credentials',
        			serverUrl: 'https://akslab-8b97af61.hcp.eastasia.azmk8s.io:443') {

        			// sh "kubectl set image $K8S_NAMESPACE/$PROJECT_NAME $PROJECT_NAME=$DOCKER_REGISTRY/$PROJECT_NAME:${env.BUILD_TIMESTAMP}.${env.BUILD_ID}"

                    sh """
                    sed \'s#{{ docker_image }}#${DOCKER_IMAGE}#g\' deploy/app-update-deploy.tpl.yaml > deploy/update-deploy.yaml
        			kubectl apply -f deploy/update-deploy.yaml -n ${K8S_NAMESPACE}
        			"""
        		}
        	}
    }
}
