pipeline {
    agent any

    environment {
        PROMETHEUS_CONFIG_DIR = "${WORKSPACE}/prometheus"
    }

    stages {
        stage('Checkout Repo') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/manjukolkar/Monitoring-setup.git'
            }
        }

        stage('Run Node Exporter') {
            steps {
                sh '''
                docker ps -q --filter "name=node-exporter1" | grep -q . && docker stop node-exporter1 && docker rm node-exporter1 || true

                docker run -d \
                  --name=node-exporter1 \
                  -p 9100:9100 \
                  prom/node-exporter1
                '''
            }
        }

        stage('Run Prometheus') {
            steps {
                sh '''
                mkdir -p $PROMETHEUS_CONFIG_DIR
                cp prometheus.yml $PROMETHEUS_CONFIG_DIR/prometheus.yml

                docker ps -q --filter "name=prometheus" | grep -q . && docker stop prometheus && docker rm prometheus || true

                docker run -d \
                  --name=prometheus \
                  -p 9090:9090 \
                  -v $PROMETHEUS_CONFIG_DIR/prometheus.yml:/etc/prometheus/prometheus.yml \
                  prom/prometheus
                '''
            }
        }

        stage('Run Grafana') {
            steps {
                sh '''
                docker ps -q --filter "name=grafana" | grep -q . && docker stop grafana && docker rm grafana || true

                docker run -d \
                  --name=grafana \
                  -p 3000:3000 \
                  grafana/grafana
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Monitoring stack deployed successfully!"
            echo "Grafana → http://<EC2_PUBLIC_IP>:3000"
            echo "Prometheus → http://<EC2_PUBLIC_IP>:9090"
            echo "Node Exporter → http://<EC2_PUBLIC_IP>:9100/metrics"
        }
        failure {
            echo "❌ Deployment failed. Check Jenkins logs."
        }
    }
}
