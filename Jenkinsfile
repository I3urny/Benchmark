// SpotBugs
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }
        stage('Download SpotBugs and FindSecBugs Plugin') {
            steps {
                // Download SpotBugs 
                sh 'wget -nc https://github.com/spotbugs/spotbugs/releases/download/4.2.2/spotbugs-4.2.2.zip && unzip -n spotbugs-4.2.2.zip && chmod +x spotbugs-4.2.2/bin/spotbugs'
                // Download FindSecBugs Plugin
                sh 'wget -nc --content-disposition https://search.maven.org/remotecontent?filepath=com/h3xstream/findsecbugs/findsecbugs-plugin/1.11.0/findsecbugs-plugin-1.11.0.jar -P spotbugs-4.2.2/plugin/'
            }    
        }
        stage('SAST - SpotBugs with FindSecBugs Plugin') {
            steps  {
                // Execute spotbugs 
                //  -textui for cli only
                //  -effort:max to enable analyses that increase precision 
                //  -low to report all bugs
                //  -xml to set output format to xml
                sh 'spotbugs-4.2.2/bin/spotbugs -textui -effort:max -low -xml:withMessages -output spotbugs-with-findsecbugs.xml target/classes'
            }
        }
        stage('Create Benchmark results') {
            steps {
                // remove content from results/
                sh 'rm results/*'
                
                // move .xml to results/
                sh 'mv spotbugs-with-findsecbugs.xml results/'
                
                // create Scorecards
                sh './createScorecards.sh'
                
                archiveArtifacts artifacts: 'results/**.* , scorecard/**.*'
            }
        }
    }
}