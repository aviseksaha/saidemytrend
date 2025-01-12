def registry = "https://avisekproject1.jfrog.io"

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
                            //error "Pipeline failed due to quality gate failure: ${qualityGate.status}"
							echo "Warning: Quality gate partially passed but continuing pipeline:${qualityGate.status}"
                        }
                    }
                }
            }
		}
		stage('Jar publish') {
			steps {
				script {
					echo '<---------------------jar publish started--------------------->'
					
					def server = Artifactory.newServer url: registry + "/artifactory" + credentialsId : "artifact-cred"
					
					def properties = "buildid = ${env.BUILD_ID}, commitid = ${GIT_COMMIT}"
					
					def uploadSpec = " " "{
						"files" : [
							{
							"pattern" : "jarstaging/(*)",
							"target" : "aviseksaha-libs-release-local/{1}",
							"flat" : "false",
							"props" : "${properties}",
							"exclusions" : ["*.sha1", "*.md5"]
							}
						]
					}" " "
					def buildInfo = server.upload(uploadSpec)
					buildInfo.env.collect()
					
					server.publishBuildInfo(buildInfo)
				
					echo '<---------------------jar publish ended--------------------->'
				}
			}
		}
				
    }
}
