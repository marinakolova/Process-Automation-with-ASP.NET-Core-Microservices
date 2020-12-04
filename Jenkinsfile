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
  }
}
