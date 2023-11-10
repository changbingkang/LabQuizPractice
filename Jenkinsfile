pipeline {
    agent none
    stages {
        stage('Checkout SCM') {
            agent any
            steps {
                git branch: 'main', url: 'https://github.com/changbingkang/LabQuizPractice.git'
            }
        }

        stage('OWASP DependencyCheck') {
            agent any
            steps {
                dependencyCheck additionalArguments: '--format HTML --format XML', odcInstallation: 'OWASP Dependency-Check Vulnerabilities'
				dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }
        }

		// stage('SonarQube Analysis') {
		// 	agent any
        //     steps {
        //         script {
        //             def scannerHome = tool 'SonarQube'
        //             withSonarQubeEnv('SonarQube') {
        //                 sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=LabTestPractice -Dsonar.sources=."
        //             }
        //         }
        //     }

        //     post {
        //         always {
        //             recordIssues enabledForFailure: true, tool: sonarQube()
        //         }
        //     }
        // }

		stage('SonarQube Analysis') {
				agent any
				steps {
					script {
						def scannerHome = tool 'SonarQube'
						withSonarQubeEnv('SonarQube') {
							sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=LabTestPractice -Dsonar.sources=."
						}
					}
					
					// Post-build action within the stage
					script {
						recordIssues enabledForFailure: true, tool: sonarQube()
					}
				}
			}

        stage('Integration UI Test') {
            parallel {
                stage('Deploy') {
                    agent any
                    steps {
                        sh './jenkins/scripts/deploy.sh'
                        input message: 'Finished using the web site? (Click "Proceed" to continue)'
                        sh './jenkins/scripts/kill.sh'
                    }
                }
                stage('Headless Browser Test') {
                  steps {
                        script {
                            docker.image('maven:3-alpine').inside {
                                sh 'mvn -B -DskipTests clean package'
                                sh 'mvn test'
                            }
                        }
                    }
                    post {
                        always {
                            junit 'target/surefire-reports/*.xml'
                        }
                    }
                }
            }
        }



      
    }
}
