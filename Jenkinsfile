// pan-service/release/1.0/jenkins/Jenkinsfile
pipeline {
    agent any
    stages {
        stage('Delegate') {
            steps {
                git url: 'https://github.com/vinay2727/jenkins', branch: 'main'
                build job: 'jenkins/Jenkinsfile'
            }
        }
    }
}
