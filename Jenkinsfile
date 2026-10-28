
pipeline {
    agent any

    stages {
        stage('Git Clone') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/dhyy0808/spring-petclinic.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Image Create') {
            steps {
                sh '''
                    docker build -t spring-petclinic:${BUILD_NUMBER} .
                    docker tag spring-petclinic:${BUILD_NUMBER} dhyy88/spring-petclinic:${BUILD_NUMBER}
                    docker tag spring-petclinic:${BUILD_NUMBER} dhyy88/spring-petclinic:latest
                '''
            }
        }

        stage('Docker Image Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerCredentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_TOKEN'
                )]) {
                    sh '''
                        echo "$DOCKER_TOKEN" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push dhyy88/spring-petclinic:${BUILD_NUMBER}
                        docker push dhyy88/spring-petclinic:latest
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl apply -n was -f k8s/petclinic.yml
                    kubectl set image deployment/petclinic \
                        workload=dhyy88/spring-petclinic:${BUILD_NUMBER} -n was
                    kubectl rollout status deployment/petclinic -n was --timeout=180s
                '''
            }
        }
    }
}
