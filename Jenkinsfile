def img

pipeline {
    environment {
        registry = "https://github.com/JaiswalRohit13/Emp-Portal-Devops-Project/tree/dev"
        registryCredential = 'DOCKERHUB'
        githubCredential = 'GitHub-Creds'
        dockerImage = ''
        scannerHome = tool 'sonar4.8'
    }

    agent any

    parameters {
        choice(
            choices: ['Dev', 'Prod'],
            description: 'Select the target cluster',
            name: 'TARGET_CLUSTER'
        )
    }

    stages {
        stage('Checkout project') {
            steps {
                script {
                    // Checkout the project from GitHub
                    git branch: 'main',
                    credentialsId: githubCredential,
                    url: 'https://github.com/JaiswalRohit13/Emp-Portal-Devops-Project/tree/dev'
                }
            }
        }

        stage('Installing packages') {
            steps {
                script {
                    // Install required Python packages
                    sh 'pip install -r requirements.txt'
                }
            }
        }

        stage('Static Code Checking') {
            steps {
                script {
                    // Run pylint on Python files and generate a report
                    sh 'find . -name \\*.py | xargs pylint -f parseable | tee pylint.log'
                    recordIssues(
                        tool: pyLint(pattern: 'pylint.log'),
                        unstableTotalHigh: 100
                    )
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('sonar') {
                        // Run SonarQube scanner for code analysis
                        sh '''${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=DevOps-Project \
                            -Dsonar.sources=.'''
                    }
                }
            }
        }
    }
}
