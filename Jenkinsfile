pipeline {
    agent any

    environment {
        PROMETHEUS_CONFIG_DIR = "${WORKSPACE}/prometheus"
    }

    stages {
        stage('Clean Existing Containers') {
            steps {
                sh '''
                docker rm -f prometheus grafana node-exporter || true
                '''
            }
        }

        stage('Prepare Prometheus Config') {
            steps {
                sh '''
                mkdir -p $PROMETHEUS_CONFIG_DIR
                cp prometheus.yml $PROMETHEUS_CONFIG_DIR/prometheus.yml
                '''
            }
        }

        stage('Run Node Exporter') {
            steps {
                sh '''
                docker run -d \
                    --name node-exporter \
                    -p 9100:9100 \
                    --restart always \
                    prom/node-exporter
                '''
            }
        }

        stage('Run Prometheus') {
            steps {
                sh '''
                docker run -d \
                    --name prometheus \
                    -p 9090:9090 \
                    -v $PROMETHEUS_CONFIG_DIR:/etc/prometheus \
                    prom/prometheus
                '''
            }
        }

        stage('Run Grafana') {
            steps {
                sh '''
                docker volume create grafana-storage || true

                docker run -d \
                    --name grafana \
                    -p 3000:3000 \
                    -v grafana-storage:/var/lib/grafana \
                    grafana/grafana
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Monitoring stack is up!"
            echo "🔗 Grafana:     http://<your-ec2-ip>:3000"
            echo "🔗 Prometheus:  http://<your-ec2-ip>:9090"
        }
        failure {
            echo "❌ Something went wrong. Check the pipeline logs."
        }
    }
}
