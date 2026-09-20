pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    environment {
        JAVA_HOME = '/opt/java8'
        PATH = "/opt/java8/bin:/mnt/c/apache-maven-3.9.12/bin:/usr/bin:/bin"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
    }
}
