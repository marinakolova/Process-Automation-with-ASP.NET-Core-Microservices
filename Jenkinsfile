pipeline {
	agent any
	stages {
		stage('Verify Branch') {
			steps {
				echo "$GIT_BRANCH"
			}
		}
		stage('Echo the branch') {
			steps {
				echo "$GIT_BRANCH"
			}
		}
		stage('Run Unit Tests') {
			steps {
				powershell(script: """ 
				cd Server
				dotnet test
				cd ..
				""")
			}
		}
		stage('Docker Build') {
			steps {
				powershell(script: 'docker-compose build') 
				powershell(script: 'docker build -t mkolova/carrentalsystem-user-client-development --build-arg configuration=development ./Client')
				powershell(script: 'docker build -t mkolova/carrentalsystem-user-client-production --build-arg configuration=production ./Client')
				powershell(script: 'docker images -a')
			}
		}
		stage('Run Test Application') {
			steps {
				powershell(script: 'docker-compose up -d')    
			}
		}
		stage('Run Integration Tests') {
			steps {
				powershell(script: './Tests/ContainerTests.ps1') 
			}
		}
		stage('Stop Test Application') {
			steps {
				powershell(script: 'docker-compose down') 
				// powershell(script: 'docker volumes prune -f')
			}
			post {
				success {
					echo "Build successfull! You should deploy! :)"
				}
				failure {
					echo "Build failed! You should receive an e-mail! :("
				}
			}
		}
		stage('Push Images') {
			steps {
				script {
					docker.withRegistry('https://index.docker.io/v1/', 'DockerHub') {
						def image = docker.image("mkolova/carrentalsystem-identity-service")
						image.push("1.0.${env.BUILD_ID}")
						image.push('latest')
						image = docker.image("mkolova/carrentalsystem-dealers-service")
						image.push("1.0.${env.BUILD_ID}")
						image.push('latest')
						image = docker.image("mkolova/carrentalsystem-statistics-service")
						image.push("1.0.${env.BUILD_ID}")
						image.push('latest')
						image = docker.image("mkolova/carrentalsystem-notifications-service")
						image.push("1.0.${env.BUILD_ID}")
						image.push('latest')
						image = docker.image("mkolova/carrentalsystem-user-client")
						image.push("1.0.${env.BUILD_ID}")
						image.push('latest')
						image = docker.image("mkolova/carrentalsystem-admin-client")
						image.push("1.0.${env.BUILD_ID}")
						image.push('latest')
						image = docker.image("mkolova/carrentalsystem-watchdog-service")
						image.push("1.0.${env.BUILD_ID}")
						image.push('latest')
					}
				}
			}
		}
		stage('Deploy Development') {
			steps {
				withKubeConfig([credentialsId: 'DevelopmentServer', serverUrl: 'https://34.123.37.162']) {
					powershell(script: 'kubectl apply -f ./.k8s/.environment/development.yml') 
					powershell(script: 'kubectl apply -f ./.k8s/databases') 
					powershell(script: 'kubectl apply -f ./.k8s/event-bus') 
					powershell(script: 'kubectl apply -f ./.k8s/web-services') 
					powershell(script: 'kubectl apply -f ./.k8s/clients') 
					powershell(script: 'kubectl set image deployments/user-client user-client=mkolova/carrentalsystem-user-client-development:latest')
				}
			}
		}
		stage('Test Deplopment') {
			steps {
				powershell(script: './Tests/ContainerTestsDev.ps1')
			}
		}
		stage('Ask permission') {
			when { branch 'origin/main' }
			steps {
				script{
					try {
						timeout(time: 60, unit: 'SECONDS') {
							input message: 'Do you want to release this build?',
							parameters: [[$class: 'BooleanParameterDefinition',
									defaultValue: false,
									description: 'Ticking this box will do a release',
									name: 'Release']]
						}
					} 
					catch (err) {
						def user = err.getCauses()[0].getUser()
						echo 'Aborted by:\n ${user}'
					}
				}
			}
		}
		stage('Deploy Production') {
			when { branch 'origin/main' }
			steps {
				withKubeConfig([credentialsId: 'ProductionServer', serverUrl: 'https://34.67.141.88']) {
					powershell(script: 'kubectl apply -f ./.k8s/.environment/production.yml') 
					powershell(script: 'kubectl apply -f ./.k8s/databases') 
					powershell(script: 'kubectl apply -f ./.k8s/event-bus') 
					powershell(script: 'kubectl apply -f ./.k8s/web-services') 
					powershell(script: 'kubectl apply -f ./.k8s/clients') 
					powershell(script: 'kubectl set image deployments/user-client user-client=mkolova/carrentalsystem-user-client-production:latest')
				}
			}
		}
	}
}