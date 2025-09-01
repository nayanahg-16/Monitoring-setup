pipeline {
    agent any

    environment {
        PROMETHEUS_CONFIG_DIR = "${WORKSPACE}/prometheus"
        ALERTMANAGER_CONFIG_DIR = "${WORKSPACE}/alertmanager"
    }

    stages {
        stage('Clean Existing Containers') {
            steps {
                sh '''
                docker rm -f prometheus grafana node-exporter alertmanager || true
                '''
            }
        }

        stage('Prepare Config Files') {
            steps {
                sh '''
                mkdir -p $PROMETHEUS_CONFIG_DIR
                cp prometheus.yml $PROMETHEUS_CONFIG_DIR/prometheus.yml
                cp alert.rules.yml $PROMETHEUS_CONFIG_DIR/alert.rules.yml

                mkdir -p $ALERTMANAGER_CONFIG_DIR
                cp alertmanager/config.yml $ALERTMANAGER_CONFIG_DIR/config.yml
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

        stage('Run Alertmanager') {
            steps {
                sh '''
                docker run -d \
                    --name alertmanager \
                    -p 9093:9093 \
                    -v $ALERTMANAGER_CONFIG_DIR:/etc/alertmanager \
                    prom/alertmanager \
                    --config.file=/etc/alertmanager/config.yml
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
                    --link alertmanager:alertmanager \
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
            echo "✅ Monitoring stack deployed"
            echo "🔗 Grafana: http://<your-ec2-ip>:3000"
            echo "🔗 Prometheus: http://<your-ec2-ip>:9090"
            echo "🔗 Alertmanager: http://<your-ec2-ip>:9093"
        }
    }
}
