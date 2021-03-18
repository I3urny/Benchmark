// IAST
pipeline {
    agent any
    stages {
        stage('Build & Deploy') {
            steps {
                sh 'wget -nc --content-disposition https://search.maven.org/remotecontent?filepath=com/contrastsecurity/contrast-agent/3.8.2.19027/contrast-agent-3.8.2.19027.jar -P tools/Contrast/'
                sh 'cp -n tools/Contrast/contrast-agent-3.8.2.19027.jar tools/Contrast/contrast.jar'
                
                // run cargo:run and wait for Tomcat to be up and running
                sh 'mvn clean package cargo:run -Pdeploywcontrast > tomcat.log &'
                sh '( tail -f -n0 tomcat.log & ) | grep -q "Press Ctrl-C to stop the container"'
                
            }
        }
        stage('IAST - Contrast Assess CE') {
            steps {
                // run a simple request against every page
                sh './runCrawler.sh'
            }
        }
        stage('Create Benchmark results') {
            steps {
                // remove content from results/
                sh 'rm results/*'
                
                // move .log to results/
                sh 'mv tools/Contrast/working/contrast.log results/'
                
                // create Scorecards
                sh './createScorecards.sh'
                
                archiveArtifacts artifacts: 'results/**.* , scorecard/**.*'
            }
        }
    }
}