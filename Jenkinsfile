pipeline {
    agent any
	
	environment{
		PATH = "/opt/maven/bin:$PATH"
	}
	
	
    stages {
		
		stage('build') {
            steps {
                sh 'mvn clean deploy'
            }
        }
		stage('SonarQube analysis') {
			environment{
			scannerHome = tool 'saidemy-sonar-scanner'
		}
			steps {
                withSonarQubeEnv('saidemy-sonarqube-server') {
					sh '${scannerHome}/bin/sonar-scanner'
				}
			}
		}
		stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') { 
                    script {
                        def qualityGate = waitForQualityGate() 
                        if (qualityGate.status != 'OK') {
                            error "Pipeline failed due to quality gate failure: ${qualityGate.status}"
                        }
                    }
                }
            }
		}
    }
}
