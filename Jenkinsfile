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
                    // Checkout the repository
                    checkout scm

                    // Set the remote URL to HTTPS and use the GitHub token
                    sh """
                    git remote set-url origin https://vinnu2251:${GITHUB_TOKEN}@github.com/vinnu2251/go-web-app.git
                    git config --global user.email "vinaychowdarychitturi@gmail.com"
                    git config --global user.name "vinay chitturi"
                    """
                }
            }
        }

        stage('Build') {
            steps {
                sh "go build -o go-web-app"
            }
        }

        stage('Unit Test') {
            steps {
                sh "go test ./..."
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
                    sh """
                    docker build -t vinay7944/go-web-app:${commitId} .
                    """
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
                        branchName = 'main' // Default branch name if HEAD is detected
                    }
                    echo "Current branch: ${branchName}"

                    sh """
                    sed -i 's/tag: .*/tag: "${commitId}"/' helm/go-web-app-chart/values.yaml
                    git add helm/go-web-app-chart/values.yaml
                    git commit -m "Update tag in Helm chart with commit ID ${commitId}"
                    git pull origin ${branchName} --rebase
                    git push https://vinnu2251:${GITHUB_TOKEN}@github.com/vinnu2251/go-web-app.git ${branchName} -o ci.skip
                    """
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
