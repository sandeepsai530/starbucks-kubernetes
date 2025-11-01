pipeline {
    agent any
    tools {
        jdk 'JDK'
        nodejs 'NodeJS'
    }
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('GIT-checkout') {
            steps {
                git branch: 'main', credentialsId: 'github-token', url: 'https://github.com/sandeepsai530/starbucks-kubernetes.git'  
            }
        }
        stage('sonarqube analysis') {
            steps {
                withSonarQubeEnv('sonar-scanner') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=starbucks \
                    -Dsonar.projectKey=starbucks '''
                }
            }
        }
        stage('quality gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('install dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('Trivy FS scan') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage('Docker build & push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t starbucks ."
                        sh "docker tag starbucks sandeep530/starbucks:latest"
                        sh "docker push sandeep530/starbucks:latest"
                    }
                }
            }
        }
        stage('TRIVY') {
            steps {
                sh "trivy image sandeep530/starbucks:latest > trivyimage.txt"
            }
        }
        stage("App deploy to docker container") {
            steps {
                sh 'docker run -d --name starbucks -p 3000:3000 sandeep530/starbucks:latest'
            }
        }
        stage("deploy to EKS cluster") {
            steps {
                sh '''
                echo "Verifying AWS credentials..."
                aws sts get-caller-identity

                echo "Configuring kubectl for EKS cluster..."
                aws eks update-kubeconfig --region us-east-1 --name my-eks-cluster

                echo "Verifying kubeconfig..."
                kubectl config view
                
                echo "deploying applications to EKS" 
                kubectl apply -f kubernetes/manifest.yml
                '''
            }
        }
        
        
    }
}
