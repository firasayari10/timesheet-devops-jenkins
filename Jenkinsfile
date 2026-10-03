pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    environment {
        JAVA_HOME = '/opt/java8'
        PATH = "/opt/java8/bin:/mnt/c/apache-maven-3.9.12/bin:/usr/bin:/bin"

        REGISTRY = 'localhost:5000'
        IMAGE_NAME = 'backend-app'
        IMAGE_TAG = 'latest'
        REGISTRY_CREDENTIALS = 'docker-registry-credentials'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                        -t ${IMAGE_NAME}:${IMAGE_TAG} \
                        .
                '''
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "${REGISTRY_CREDENTIALS}",
                        usernameVariable: 'REGISTRY_USER',
                        passwordVariable: 'REGISTRY_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$REGISTRY_PASSWORD" | docker login ${REGISTRY} \
                            -u "$REGISTRY_USER" \
                            --password-stdin

                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} \
                            ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}

                        docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}

                        docker logout ${REGISTRY}
                    '''
                }
            }
        }

        stage('Deploy MySQL') {
            steps {
                sh '''
                    docker network inspect timesheet-network >/dev/null 2>&1 || \
                        docker network create timesheet-network

                    docker rm -f mysql 2>/dev/null || true

                    docker run -d \
                        --name mysql \
                        --network timesheet-network \
                        -e MYSQL_ROOT_PASSWORD= \
                        -e MYSQL_ALLOW_EMPTY_PASSWORD=yes \
                        -e MYSQL_DATABASE=timesheet-devops-db \
                        mysql:8

                    sleep 15
                '''
            }
        }

        stage('Deploy Backend') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "${REGISTRY_CREDENTIALS}",
                        usernameVariable: 'REGISTRY_USER',
                        passwordVariable: 'REGISTRY_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$REGISTRY_PASSWORD" | docker login ${REGISTRY} \
                            -u "$REGISTRY_USER" \
                            --password-stdin

                        docker rm -f backend-app 2>/dev/null || true

                        docker pull ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}

                        docker run -d \
                            --name backend-app \
                            --network timesheet-network \
                            -p 8082:8082 \
                            -e SPRING_DATASOURCE_URL="jdbc:mysql://mysql:3306/timesheet-devops-db?useUnicode=true&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=UTC" \
                            -e SPRING_DATASOURCE_USERNAME=root \
                            -e SPRING_DATASOURCE_PASSWORD= \
                            ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}

                        docker logout ${REGISTRY}
                    '''
                }
            }
        }

        stage('Verification') {
            steps {
                sh '''
                    sleep 5
                    docker ps
                    docker logs backend-app
                '''
            }
        }
    }
}
