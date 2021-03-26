// ZAP
pipeline {
    agent any
    stages {
        stage('Build & Deploy') {
            steps {
                sh 'wget -nc --content-disposition https://github.com/zaproxy/zaproxy/releases/download/v2.10.0/ZAP_2.10.0_Linux.tar.gz'
                sh 'tar --skip-old-files -xzf  ZAP_2.10.0_Linux.tar.gz'
                sh 'chmod +x ZAP_2.10.0/zap.sh'
                
                // run cargo:run and wait for Tomcat to be up and running
                sh 'mvn clean package cargo:run -Pdeploy > tomcat.log &'
                sh '( tail -f -n0 tomcat.log & ) | grep -q "Press Ctrl-C to stop the container"'
                
            }
        }
        stage('DAST - OWASP Zed Attack Proxy') {
            steps {
                sh '''
                    ZAP_2.10.0/zap.sh -cmd \
                    -config spider.thread=12 \
                    -config scanner.threadPerHost=12 \
                    -config scanner.injectable=15 \
                    -config scanner.scanHeadersAllRequests=true \
                    -config alert.maxInstances=50000 \
                    -quickurl https://localhost:8443/benchmark/ \
                    -quickprogress \
                    -quickout $(pwd)/zap.xml
                '''
            }
        }
        stage('Create Benchmark results') {
            steps {
                // remove content from results/
                sh 'rm results/*'
                
                // move .log to results/
                sh 'mv zap.xml results/'
                
                // create Scorecards
                sh './createScorecards.sh'
                
                archiveArtifacts artifacts: 'results/**.* , scorecard/**.*'
            }
        }
    }
}