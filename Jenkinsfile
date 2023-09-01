pipeline {
    agent any
    tools{
        maven 'MAVEN3'
    }
    environment {
        registry = '426527747391.dkr.ecr.us-east-1.amazonaws.com/hellodatarepo'
        registryCredential = 'awscreds'
        dockerimage = ''
        cluster = "vprofile"
        service = "vprofileappsvc"
    }
    stages{
        stage("Checkout the project") {
           steps{
              git branch: 'docker', url: 'https://github.com/devopshydclub/vprofile-project.git'
           } 
        }
        stage("Build the package"){
            steps {
                sh 'mvn clean package'
            }
        }
		stage("Sonar Quality Check"){
		steps{
		    script{
		     withSonarQubeEnv(installationName: 'sonar-10', credentialsId: 'jenkins-token') {
		     sh 'mvn sonar:sonar'
	    	}
	    	 timeout(time: 1, unit: 'HOURS') {
              def qg = waitForQualityGate()
              if (qg.status != 'OK') {
                  error "Pipeline aborted due to quality gate failure: ${qg.status}"
         }
	    	 }
		    }
      }
    }	
    stage('Building the Image') {
        steps {
            script {
            dockerImage = docker.build( registry + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
        }
    }
    }

    stage ('Deploy the Image to Amazon ECR') {
       steps {
           script {
                docker.withRegistry("http://" + registry, "ecr:us-east-1:" + registryCredential ) {
                dockerImage.push("$BUILD_NUMBER")
                dockerImage.push('latest')
            }
         }
      }
    }

    // Deploy the image to ECS
      stage('Deploy to ecs') {
          steps {
        withAWS(credentials: 'awscreds', region: 'us-east-1') {
          sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
        }
      }
     }

  }
}
