pipeline {
    agent  { label 'wsl'
    }
    stages {
        stage('Checkout') {
            steps {
            git 'https://github.com/banbi95/vhr.git'
            }
            
        }

        stage('build') {
            steps {
                sh 'cd vhr/vhr && docker build -t banbi95/vhr-backend:1.1 .'
                
            }
        }

    }
}