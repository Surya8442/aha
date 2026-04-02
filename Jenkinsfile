
pipeline {
    agent any

    environment {
        SONARQUBE_ENV = 'sq'
        DOCKER_IMAGE = "surya8442/aha"
        
        AWS_DEFAULT_REGION = 'ap-south-1'
        RECIPIENTS = 'suryakandipalli@gmail.com'
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/Surya8442/aha.git'
            }
        }
        stage('BUILD') {
        steps {
            sh 'mvn clean package -DskipTests'
        }
    }
        stage('JENKINS TO NEXUS') {
        steps {
          withMaven(jdk: 'jdk21', maven: 'maven3', traceability: true) {
             sh 'mvn deploy'
}
        }
    }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }


        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:latest .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Docker-cred', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push $DOCKER_IMAGE:latest
                    docker logout
                    '''
                }
            }
        }
         stage('Build') {
            steps {
                echo "Building..."
            }
        }

        stage('Test') {
            steps {
                echo "Testing..."
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    export KUBECONFIG=/var/lib/jenkins/.kube/config
                    kubectl get nodes
                aws eks update-kubeconfig --region ap-south-1 --name mycluster
                kubectl apply -f deployment.yml
                kubectl apply -f service.yml
                '''
            }
        }
    }

    post {
        success {
            emailext(
                subject: "Jenkins Job '${env.JOB_NAME}' Success",
                body: "Good news! Job '${env.JOB_NAME}' (#${env.BUILD_NUMBER}) succeeded.\n\nCheck console output at ${env.BUILD_URL}",
                to: "${RECIPIENTS}"
            )
        }

        failure {
            emailext(
                subject: "Jenkins Job '${env.JOB_NAME}' Failed",
                body: "Alert! Job '${env.JOB_NAME}' (#${env.BUILD_NUMBER}) failed.\n\nCheck console output at ${env.BUILD_URL}",
                to: "${RECIPIENTS}"
            )
        }

    }
}
