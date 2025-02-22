pipeline {
    agent  { label 'wsl'
    }
    environment {
        DOCKER_REGISTRY = 'demo.goharbor.io' 
        DOCKER_IMAGE = 'vhr/vhr-backend'
        VHRBACKEND_TAG = 'latest'
        // define credentials in jenkins server
        DOCKER_USERNAME=credentials('harbor-username')  
        DOCKER_PASSWORD=credentials('harbor-password')
    }
    stages {
        stage('Checkout') {
            steps {
            git 'https://github.com/banbi95/vhr.git'
            }
            
        }
           stage('get latest tag'){
            steps {
                script {
                    try {
                        sh 'git fetch --tags'
                        env.VHRBACKEND_TAG = sh(script: 'git describe --tags $(git rev-list --tags --max-count=1) || echo "v0.0.0"', returnStdout: true).trim()
                        if (env.VHRBACKEND_TAG == 'v0.0.0') {
                            error("No valid tags found in the repository")
                        }
                    } catch (err) {
                        currentBuild.result = 'FAILED'
                        echo "Failed to get latest tag: ${err}"
                        throw err
                    }
                }
            }
           }

        stage('build') {
            steps {
                sh 'cd vhr/vhr && docker build -t ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:latest .'
                
            }
        }

    stage('push') {
            steps {
                sh """ docker tag ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:latest ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${env.VHRBACKEND_TAG} 
                       docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD} ${DOCKER_REGISTRY} 
                       docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:latest 
                       docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${env.VHRBACKEND_TAG}
                    """ 
  
            }
        }

    // }
    stage('deploy') {
        agent {
            label 'wsl'
        }
        steps {
             sh  ''' cd vhr/vhr && \
             docker pull demo.goharbor.io/vhr/vhr-backend && \
             docker-compose up -d 
            '''

        }
       
    }
}
