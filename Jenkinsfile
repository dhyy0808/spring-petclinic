
pipeline {
    agent any
    tools {
        jdk 'JDK21'
        maven 'M3'
    }
    environment {
        DOCKERHUB_CRED = credentials('dockerCredentials')
    }

    stages {
        // Github로부터 소스코드 다운로드
        stage('Git Clone') {
            steps {
                echo 'Git Clone'
                git url: 'https://github.com/dhyy0808/spring-petclinic.git',
                branch: 'main'
            }
        }

        // Maven을 이용한 Build
        stage('Maven Build') {
            steps {
                echo 'Maven Build'
                sh 'mvn -Dmaven.test.failure.ignore=true clean package'
            }
        }

        // Docker Image 생성
        stage('Docker Image Create') {
            steps {
                echo 'Docker Image Create'
                sh '''
                docker build -t spring-petclinic:${BUILD_NUMBER} .
                docker tag spring-petclinic:${BUILD_NUMBER} dhyy88/spring-petclinic:${BUILD_NUMBER}
                docker tag spring-petclinic:${BUILD_NUMBER} dhyy88/spring-petclinic:latest
                '''
            }
        }

        // Docker Hub Image Push
        stage('Docker Image Push') {
            steps {
                echo 'Docker Image Push'

                // Docker Login
                sh 'echo "$DOCKERHUB_CRED_PSW" | docker login -u "$DOCKERHUB_CRED_USR" --password-stdin'

                // Docker Hub Push
                sh '''
                docker image push dhyy88/spring-petclinic:${BUILD_NUMBER}
                docker image push dhyy88/spring-petclinic:latest
                docker logout
                '''
            }
        }

        // Kubernetes 자동 배포
        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploy to Kubernetes'
                sh '''
                kubectl apply -n was -f k8s/petclinic.yml
                kubectl set image deployment/petclinic workload=dhyy88/spring-petclinic:${BUILD_NUMBER} -n was
                kubectl rollout status deployment/petclinic -n was --timeout=180s
                '''
            }
        }
    }
}
