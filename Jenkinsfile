pipeline {
    agent any

    tools {
        jdk 'JDK21'
        maven 'Maven'
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

stage('Docker Build') {
    steps {
        script {
            echo 'Building Docker image'
            sh 'docker build -t spring-petclinic:latest .'
        }
    }
}


}

}

}
