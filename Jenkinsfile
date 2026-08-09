pipeline {
    agent any

    tools {
        jdk 'JDK21'
        maven 'Maven'
    }
environment {
    ACR_NAME = 'sindamaryacr2026'
    ACR_LOGIN_SERVER = 'sindamaryacr2026.azurecr.io'
    IMAGE_NAME = 'spring-petclinic'
    IMAGE_TAG = 'latest'
    TENANT_ID = '091502bd-d7d4-42f5-82c2-eb0671a586a8'
    SUBSCRIPTION_ID = '669b4ff7-c7c8-4658-9adb-729a158e4fdb'
}



    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Maven Validate') {
            steps {
                sh 'mvn validate'
            }
        }

        stage('Maven Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Maven Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Maven Package') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarCloud') {
                    sh '''
                        mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
                        -Dsonar.projectKey=sindamary_spring-petclinic \
                        -Dsonar.organization=sindamary
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    echo 'Building Docker image'
                    sh 'docker build -t spring-petclinic:latest .'
                }
            }
        }
stage('Trivy Image Scan') {
    steps {
        echo 'Trivy image scan started'
        sh '''
            trivy image \
            --severity HIGH,CRITICAL \
            --format table \
            --output trivy-image-report.txt \
            spring-petclinic:latest
        '''
        echo 'Trivy image scan completed'
    }
}
stage('Azure Login') {
    steps {
        withCredentials([
            usernamePassword(
                credentialsId: 'azure-acr-spn',
                usernameVariable: 'AZURE_CLIENT_ID',
                passwordVariable: 'AZURE_CLIENT_SECRET'
            )
        ]) {
            sh '''
                az login --service-principal \
                  --username "$AZURE_CLIENT_ID" \
                  --password "$AZURE_CLIENT_SECRET" \
                  --tenant "$TENANT_ID"

                az account set --subscription "$SUBSCRIPTION_ID"
            '''
        }
    }
}

stage('ACR Login') {
    steps {
        sh '''
            az acr login --name "$ACR_NAME"
        '''
    }
}

stage('Docker Push to ACR') {
    steps {
        sh '''
            docker tag \
              ${IMAGE_NAME}:${IMAGE_TAG} \
              ${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG}

            docker push \
              ${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG}
        '''
    }
}
stage('Connect to AKS') {
    steps {
        sh '''
            az aks get-credentials \
              --resource-group rg-project4-jenkins \
              --name springpetclinicaks \
              --overwrite-existing
        '''
    }
}

stage('Deploy to Kubernetes') {
    steps {
        sh '''
            kubectl apply -f k8s/db.yml
            kubectl apply -f k8s/petclinic.yml
        '''
    }
}

stage('Verify Deployment') {
    steps {
        sh '''
            kubectl rollout status deployment/petclinic --timeout=180s
            kubectl get pods
            kubectl get svc
            kubectl get deployments
        '''
    }
}

    }
}
