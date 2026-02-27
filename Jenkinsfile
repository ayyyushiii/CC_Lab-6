pipeline {
    agent any

    stages {

        stage('Build Backend Image') {
            steps {
                sh '''
                docker rmi -f backend-app || true
                docker build -t backend-app backend
                '''
            }
        }

        stage('Deploy Backend Containers') {
            steps {
                sh '''
                # Create custom network (if not exists)
                docker network create lab-network || true

                # Remove old containers
                docker rm -f backend1 backend2 || true

                # Run backends inside custom network
                docker run -d --name backend1 --network lab-network backend-app
                docker run -d --name backend2 --network lab-network backend-app

                sleep 5
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                docker rm -f nginx-lb || true

                # Run nginx inside SAME network
                docker run -d --name nginx-lb \
                    --network lab-network \
                    -p 80:80 nginx

                sleep 3

                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                sleep 2

                docker exec nginx-lb nginx -s reload
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline failed. Check console logs."
        }
    }
}