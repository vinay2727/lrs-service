pipeline {
    agent any
    stages {
        stage('Run Central Pipeline') {
            steps {
                git url: 'https://github.com/vinay2727/jenkins.git', branch: 'main'
                script {
                    def central = load 'centralPipeline.groovy'
                    central()
                }
            }
        }
    }
}
