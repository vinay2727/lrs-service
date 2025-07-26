pipeline {
    agent any

    environment {
        KUBECONFIG = '/c/Users/rbih4/.kube/config'  // Adjust as per Jenkins agent
    }

    parameters {
        choice(name: 'ENVIRONMENT', choices: ['qa', 'prod'], description: 'Choose environment to deploy')
    }

    stages {
        stage('Extract Metadata') {
            steps {
                script {
                    def jobParts = env.JOB_NAME.split('/')
                    env.REPO_NAME = jobParts[1]
                    env.BRANCH_NAME = jobParts[2]
                    env.TAG = "${env.REPO_NAME}-${env.BUILD_NUMBER}"
                    env.DOCKER_IMAGE = "drdocker108/${env.REPO_NAME}:${env.TAG}"
                    echo "Repo: ${env.REPO_NAME}, Branch: ${env.BRANCH_NAME}, ENV: ${params.ENVIRONMENT}"
                }
            }
        }

        stage('Checkout Code') {
            steps {
                git url: "https://github.com/vinay2727/${env.REPO_NAME}.git", branch: "${env.BRANCH_NAME}"
            }
        }

        stage('Build & Push Image (QA only)') {
            when {
                expression { params.ENVIRONMENT == 'qa' }
            }
            steps {
                sh 'mvn clean package -DskipTests'
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        docker build -t ${env.DOCKER_IMAGE} .
                        echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                        docker push ${env.DOCKER_IMAGE}
                        docker logout
                    """
                }
            }
        }

        stage('Fetch Image from QA (Prod only)') {
            when {
                expression { params.ENVIRONMENT == 'prod' }
            }
            steps {
                script {
                    def imageTag = sh(
                        script: """
                            kubectl --kubeconfig=${KUBECONFIG} -n qa get pods -l app=${env.REPO_NAME} -o jsonpath='{.items[0].spec.containers[0].image}'
                        """,
                        returnStdout: true
                    ).trim()
                    env.DOCKER_IMAGE = imageTag
                    echo "Using promoted image from QA: ${env.DOCKER_IMAGE}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "Deploying to ${params.ENVIRONMENT} with image ${env.DOCKER_IMAGE}"
                }

                sh """
                    echo "Setting context"
                    kubectl --kubeconfig=${KUBECONFIG} config use-context minikube || true

                    echo "Creating namespace ${params.ENVIRONMENT} if not exists"
                    kubectl --kubeconfig=${KUBECONFIG} create namespace ${params.ENVIRONMENT} --dry-run=client -o yaml | kubectl apply -f -

                    echo "Applying deployment manifest"
                    cat <<EOF | kubectl --kubeconfig=${KUBECONFIG} apply -n ${params.ENVIRONMENT} -f -
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: ${env.REPO_NAME}
                    spec:
                      replicas: 1
                      selector:
                        matchLabels:
                          app: ${env.REPO_NAME}
                      template:
                        metadata:
                          labels:
                            app: ${env.REPO_NAME}
                        spec:
                          containers:
                          - name: ${env.REPO_NAME}
                            image: ${env.DOCKER_IMAGE}
                            ports:
                            - containerPort: 8080
                    EOF
                """
            }
        }
    }
}
