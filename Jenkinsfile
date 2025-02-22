pipeline {
    agent { label 'wsl' }
    environment {
        DOCKER_REGISTRY = 'demo.goharbor.io'
        DOCKER_IMAGE = 'vhr/vhr-backend'
        // VHRBACKEND_TAG = 'latest'  
        // define credentials in jenkins server
        DOCKER_CREDENTIALS_ID = 'harbor-credentials'
        IMAGE_EXISTS= ''
    }
    parameters {
        password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'Enter a password')
    }
    stages {
        stage('Checkout and Get newest Tag') {
            steps {
                script {
                    git 'https://github.com/banbi95/vhr.git'
                    sh 'git fetch --tags'
                    
                    // get newest tag
                    env.LATEST_TAG = sh(
                        script: '''
                            git describe --tags $(git rev-list --tags --max-count=1) || echo "v0.0.0"
                        ''',
                        returnStdout: true
                    ).trim()
                    echo "Latest tag: ${env.LATEST_TAG}"

                    if (env.LATEST_TAG == 'v0.0.0') {
                        echo "No valid tags found in the repository, using default tag 'latest'"
                        env.LATEST_TAG = 'latest'
                    }

                    // make sure VHRBACKEND_TAG is updated with latest tag  
                    env.VHRBACKEND_TAG = env.LATEST_TAG
                    echo "Updated VHRBACKEND_TAG to: ${env.VHRBACKEND_TAG}"
                }
            }
        }

        stage('check') {
            steps {
                withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    script {
                        retry(3) {
                            def imageExists = sh(
                                script: """
                                if docker manifest inspect ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${VHRBACKEND_TAG} >/dev/null 2>&1; then
                                    echo 'true'
                                else
                                    echo 'false'
                                 fi
                                    """,

                                    returnStdout: true
                            ).trim()
                            env.IMAGE_EXISTS = imageExists 
                        }
                    }
                }
            }
        }

        stage('push') {
            when {
                expression { env.IMAGE_EXISTS == "false" }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    script {
                        retry(3) {
                            sh """
                                echo "Attempting docker login"
                                echo "\$DOCKER_PASSWORD" | docker login -u "\$DOCKER_USERNAME" --password-stdin $DOCKER_REGISTRY
                                echo "Pushing image with tag latest"
                                docker push \${DOCKER_REGISTRY}/\${DOCKER_IMAGE}:latest
                                docker tag \${DOCKER_REGISTRY}/\${DOCKER_IMAGE}:latest \${DOCKER_REGISTRY}/\${DOCKER_IMAGE}:\${VHRBACKEND_TAG}
                                echo "Pushing image with tag: \${env.VHRBACKEND_TAG}"
                                docker push ${DOCKER_REGISTRY}/\${DOCKER_IMAGE}:${VHRBACKEND_TAG}
                            """
                        }
                    }
                }
            }
        }

        stage('deploy') {
            // agent {
            //     label 'wsl'
            // }
            environment { 
                VHRBACKEND_VERSION="${env.LATEST_TAG}"
                DOCKER_REGISTRY="${env.DOCKER_REGISTRY}"
                DOCKER_IMAGE="${env.DOCKER_IMAGE}"

            }
            
            steps {
                withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh """
                    #!/bin/bash
                    set -eo 
                    echo VHRBACKEND_VERSION:\${VHRBACKEND_VERSION}
                    echo "\$DOCKER_PASSWORD" | docker login -u "\$DOCKER_USERNAME" --password-stdin ${DOCKER_REGISTRY}
                    docker pull \${DOCKER_REGISTRY}/\${DOCKER_IMAGE}:\${VHRBACKEND_VERSION}
                    if [ -f dockerCompose.yaml ]; then
                        docker-compose -f dockerCompose.yaml up -d
                    else 
                        echo >&2 "dockerCompose file doesn\'t exist"
                        exit 1
                    fi
                    """
                
            }
        }
    }
}
}