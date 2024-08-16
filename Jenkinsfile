pipeline {
    agent any

    tools {
        go 'go-1.22.5'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        SONAR_TOKEN = credentials('sonar-cred')
        GITHUB_TOKEN = credentials('git-cred') // GitHub token
        DOCKER_CRED = credentials('docker-cred') // Docker credentials
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    checkout scm
                    // Configure Git
                    sh '''
                    git config --global user.email "vinaychowdarychitturi@gmail.com"
                    git config --global user.name "vinay chitturi"
                    '''
                }
            }
        }

        stage('Sync with Remote') {
            steps {
                script {
                    // Fetch the latest changes and determine the current branch
                    def branchName = sh(script: 'git rev-parse --abbrev-ref HEAD', returnStdout: true).trim()
                    if (branchName == 'HEAD') {
                        // If in detached HEAD state, switch to the main branch
                        branchName = 'main'
                        sh "git checkout ${branchName}"
                    }

                    echo "Current branch: ${branchName}"
                    sh '''
                    git fetch origin
                    git pull origin ${branchName} --rebase
                    '''
                }
            }
        }

        stage('Apply Local Changes') {
            steps {
                echo "Applying local changes (if any)"
                // Apply your local changes here if necessary
            }
        }

        stage('Commit Local Changes') {
            steps {
                script {
                    def status = sh(script: 'git status --porcelain', returnStdout: true).trim()
                    if (status) {
                        sh '''
                        git add .
                        git commit -m "Apply local changes and update repository"
                        '''
                    } else {
                        echo "No changes to commit."
                    }
                }
            }
        }

        stage('Push Changes to Remote') {
            steps {
                script {
                    def branchName = sh(script: 'git rev-parse --abbrev-ref HEAD', returnStdout: true).trim()
                    echo "Pushing changes to branch: ${branchName}"

                    // Push the changes
                    sh '''
                    git push https://vinnu2251:${GITHUB_TOKEN}@github.com/vinnu2251/go-web-app.git ${branchName}
                    '''
                }
            }
        }

        stage('Build') {
            steps {
                sh 'go build -o go-web-app'
            }
        }

        stage('Unit Test') {
            steps {
                sh 'go test ./...'
            }
        }

        stage('Run SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh "${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=gowebapp -Dsonar.projectName=gowebapp"
                }
            }
        }

        stage('Docker Build & Tag') {
            steps {
                script {
                    def commitId = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    echo "Current commit ID: ${commitId}"

                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t vinay7944/go-web-app:${commitId} ."
                    }
                }
            }
        }

        stage('Docker Push Image') {
            steps {
                script {
                    def commitId = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()

                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push vinay7944/go-web-app:${commitId}"
                    }
                }
            }
        }

        stage('Update Helm Chart Tag') {
            steps {
                script {
                    def commitId = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    echo "Current commit ID: ${commitId}"

                    def branchName = sh(script: 'git rev-parse --abbrev-ref HEAD', returnStdout: true).trim()
                    if (branchName == 'HEAD') {
                        branchName = 'main'
                    }
                    echo "Current branch: ${branchName}"

                    sh '''
                    sed -i 's/tag: .*/tag: "${commitId}"/' helm/go-web-app-chart/values.yaml
                    git add helm/go-web-app-chart/values.yaml
                    git commit -m "Update tag in Helm chart with commit ID ${commitId}"
                    git pull origin ${branchName} --rebase
                    git push https://vinnu2251:${GITHUB_TOKEN}@github.com/vinnu2251/go-web-app.git ${branchName}
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline complete"
            cleanWs() // Clean workspace after the build
        }
    }
}
