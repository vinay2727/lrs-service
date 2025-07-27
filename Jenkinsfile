node {
    stage('Checkout Central Pipeline') {
        dir('central') {
            git url: 'https://github.com/vinay2727/jenkins.git', branch: 'main', credentialsId: 'Github-PAT'
        }
    }
    stage('Delegate to Central') {
        def pipeline = load 'central/Jenkinsfile'
        pipeline()
    }
}
