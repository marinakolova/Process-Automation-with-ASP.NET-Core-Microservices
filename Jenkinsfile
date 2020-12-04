pipeline {
	agent any
	stages {
		stage('Verify Branch') {
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
						def identity-service-image = docker.image("mkolova/carrentalsystem-identity-service")
						identity-service-image.push("1.0.${env.BUILD_ID}")
						identity-service-image.push('latest')
						def dealers-service-image = docker.image("mkolova/carrentalsystem-dealers-service")
						dealers-service-image.push("1.0.${env.BUILD_ID}")
						dealers-service-image.push('latest')
						def statistics-service-image = docker.image("mkolova/carrentalsystem-statistics-service")
						statistics-service-image.push("1.0.${env.BUILD_ID}")
						statistics-service-image.push('latest')
						def notifications-service-image = docker.image("mkolova/carrentalsystem-notifications-service")
						notifications-service-image.push("1.0.${env.BUILD_ID}")
						notifications-service-image.push('latest')
						def user-client-image = docker.image("mkolova/carrentalsystem-user-client")
						user-client-image.push("1.0.${env.BUILD_ID}")
						user-client-image.push('latest')
						def admin-client-image = docker.image("mkolova/carrentalsystem-admin-client")
						admin-client-image.push("1.0.${env.BUILD_ID}")
						admin-client-image.push('latest')
						def watchdog-service-image = docker.image("mkolova/carrentalsystem-watchdog-service")
						watchdog-service-image.push("1.0.${env.BUILD_ID}")
						watchdog-service-image.push('latest')
					}
				}
			}
		}
	}
}