pipeline {

    agent any

    environment {

        SCANNER_HOME = tool 'sonar-scanner'

        IMAGE_NAME = 'ravikirankadari/food-dev'
        IMAGE_TAG  = 'v1'

    }

    stages {

        stage('Clone Code') {
            steps {

                git(
                    branch: 'main',
                    url: 'https://github.com/kadariravikiran/FOOD_DEV.git',
                    credentialsId: 'git-cred'
                )

            }
        }

        stage('Check Project Structure') {
            steps {

                sh 'pwd'
                sh 'ls -la'
                sh 'find . -name package.json'
                sh 'find . -name pom.xml'

            }
        }

        stage('Install Node Modules') {
            steps {

                sh 'npm install'

            }
        }

        stage('Build Application') {
            steps {

                sh 'npm run build'

            }
        }

        stage('SonarQube Analysis') {
            steps {

                withSonarQubeEnv('sonarqube') {

                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=food-dev \
                    -Dsonar.projectName=food-dev \
                    -Dsonar.sources=. 
                    '''

                }

            }
        }

        stage('Trivy File Scan') {
            steps {

                sh 'trivy fs .'

            }
        }

        stage('Docker Build') {
            steps {

                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'

            }
        }

        stage('Trivy Docker Image Scan') {
            steps {

                sh 'trivy image $IMAGE_NAME:$IMAGE_TAG'

            }
        }

        stage('Docker Push') {
            steps {

                withDockerRegistry(
                    credentialsId: 'docker-cred',
                    url: 'https://index.docker.io/v1/'
                ) {

                    sh 'docker push $IMAGE_NAME:$IMAGE_TAG'

                }

            }
        }

        stage('Deploy To Kubernetes') {
            steps {

                sh 'kubectl apply -f deployment.yaml'
                sh 'kubectl apply -f service.yaml'

            }
        }

    }

}
