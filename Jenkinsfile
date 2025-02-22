pipeline {
    agent { label 'wsl' }
    environment {
        DOCKER_REGISTRY = 'demo.goharbor.io'
        DOCKER_IMAGE = 'vhr/vhr-backend'
        DOCKER_CREDENTIALS_ID = 'harbor-credentials'
    }

    stages {
        stage('Checkout and Get newest Tag') {
            steps {
                script {
                    git 'https://github.com/banbi95/vhr.git'
                    sh 'git fetch --tags'

                    // 获取最新标签
                    try {
                        env.LATEST_TAG = sh(
                            script: '''
                                git describe --tags $(git rev-list --tags --max-count=1) || echo "v0.0.0"
                            ''',
                            returnStdout: true
                        ).trim()
                    } catch (Exception e) {
                        echo "获取标签失败: ${e.message}"
                        env.LATEST_TAG = 'v0.0.0'
                    }

                    if (env.LATEST_TAG == 'v0.0.0') {
                        echo "未找到有效标签，使用默认标签 latest"
                        env.LATEST_TAG = 'latest'
                    }

                    env.VHRBACKEND_TAG = env.LATEST_TAG
                    echo "更新后的镜像标签: ${env.VHRBACKEND_TAG}"
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
                                echo "登录Docker仓库"
                                echo "\$DOCKER_PASSWORD" | docker login -u "\$DOCKER_USERNAME" --password-stdin $DOCKER_REGISTRY
                                echo "推送 latest 标签镜像"
                                docker push \${DOCKER_REGISTRY}/\${DOCKER_IMAGE}:latest
                                docker tag \${DOCKER_REGISTRY}/\${DOCKER_IMAGE}:latest \${DOCKER_REGISTRY}/\${DOCKER_IMAGE}:\${VHRBACKEND_TAG}
                                echo "推送 \${VHRBACKEND_TAG} 标签镜像"
                                docker push ${DOCKER_REGISTRY}/\${DOCKER_IMAGE}:${VHRBACKEND_TAG}
                            """
                        }
                    }
                }
            }
        }

        stage('deploy') {
            environment { 
                VHRBACKEND_VERSION = "${env.LATEST_TAG}"
                DOCKER_REGISTRY = "${env.DOCKER_REGISTRY}"
                DOCKER_IMAGE = "${env.DOCKER_IMAGE}"
            }
            steps {
                withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh """
                    #!/bin/bash
                    set -eo 
                    echo "当前版本: \${VHRBACKEND_VERSION}"
                    echo "\$DOCKER_PASSWORD" | docker login -u "\$DOCKER_USERNAME" --password-stdin ${DOCKER_REGISTRY}
                    if [ -f dockerCompose.yaml ]; then
                        docker-compose -f dockerCompose.yaml pull --ignore-pull-failures
                        docker-compose -f dockerCompose.yaml up -d --force-recreate
                    else 
                        echo >&2 "错误: dockerCompose.yaml 文件不存在"
                        exit 1
                    fi
                    """
                }
            }
        }
    }
}