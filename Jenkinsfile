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

    }
}
