def img;
pipeline {
    environment {
        registry = "nitesh99sharma/emp-portal-project"
        registryCredential = 'DOCKERHUB'
        KUBECONFIG = credentials('kubeconfig-aks')
        githubCredential = 'GitHub-Creds'
        dockerImage = ''
    }
    agent any

    stages {
        stage('Checkout project') {
            steps {
                script {
                    git branch: 'main',
                    credentialsId: githubCredential,
                     url: 'https://github.com/niteshsharma99/EMP-Portal-Project-DevOps.git'
                }
            }
        }
        stage('Installing packages') {
            steps {
                script {
                    sh 'pip install -r requirements.txt'
                }
            }
        }
        stage('Static Code Checking') {
            steps {
                script {
                    sh 'find . -name \\*.py | xargs pylint -f parseable | tee pylint.log'
                    recordIssues(
                        tool: pyLint(pattern: 'pylint.log'),
                        unstableTotalHigh: 100
                    )
                }
            }
        }
        stage("Testing with pytest") {
            steps {
                script {
                    withPythonEnv('python3') {
                        sh 'pip install pytest'
                        sh 'pip install flask_sqlalchemy'
                        sh 'pytest test_app.py'
                    }
                }
            }
        }
        stage ('Clean Up') {
            steps {
                sh returnStatus: true, script: 'docker stop $(docker ps -a | grep ${JOB_NAME} | awk \'{print $1}\')'
                sh returnStatus: true, script: 'docker rmi $(docker images | grep ${registry} | awk \'{print $3}\') --force'
                sh returnStatus: true, script: 'docker rm ${JOB_NAME}'
            }
        }
        stage('Build Image') {
            steps {
                script {
                    img = registry + ":${env.BUILD_ID}"
                    println("${img}")
                    dockerImage = docker.build("${img}")
                }
            }
        }
        stage('Push To DockerHub') {
            steps {
                script {
                    docker.withRegistry( 'https://registry.hub.docker.com ', registryCredential ) {
                        dockerImage.push()
                    }
                }
            }
        }
        
        stage('Deploy') {
           steps {
                sh label: '', script: "docker run -d --name ${JOB_NAME} -p 5000:5000 ${img}"
          }
        }
        stage('Deploy to Kubernetes aks') {
                    steps {
                        script {
                            withCredentials([file(credentialsId: 'kubeconfig-aks', variable: 'KUBECONFIG')]) {
                                sh "kubectl config view --kubeconfig=$KUBECONFIG"
                                sh "kubectl get namespaces --kubeconfig=$KUBECONFIG"
                                sh "kubectl apply -f deployment.yaml --kubeconfig=$KUBECONFIG"
                                sh "kubectl apply -f service.yaml --kubeconfig=$KUBECONFIG"
                            }
                        }
                    }
                }
    }
post {
        always {
            slackSend channel: '#devops-project',
                      color: 'good',
                      message: "Build Status: ${currentBuild.currentResult}",
                      teamDomain: 'jenkinsintegr-kfn1541',
                      tokenCredentialId: 'slack-integration' // Replace with the ID of the Jenkins credential containing your Slack API token
        }
    }
}
