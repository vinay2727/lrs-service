node {
    stage('Delegate to Central') {
        git url: 'https://github.com/vinay2727/jenkins.git', branch: 'main'
        def mainPipeline = load 'Jenkinsfile'
        mainPipeline()
    }
}
