pipeline {
    agent any
    environment {

	  GIT_URL = "https://github.com/LiuYuDamon/web-front.git"
		DOCKER_REPO="registry.cn-beijing.aliyuncs.com/damondocker/web-front"
		DOCKER_REG="https://registry.cn-beijing.aliyuncs.com"
		DOCKER_REG_KEY = "b9daa4e5-8bf3-49ca-8848-2451b125e0a4"
		dockerImage = ''

    }
    stages {

    	stage('CheckOut Code'){
         	steps{
            	checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: GIT_CRED, url: GIT_URL]]])
            	}
              }
        stage('Angular Build'){
        	steps{
              sh 'mkdir -p server/public'
              sh 'npm install'
        	    sh 'ng build --prod'
        	}

        }

        stage('Building image') {
	      steps{
	        script {
	           docker.withRegistry( DOCKER_REG, DOCKER_REG_KEY ) {dockerImage = docker.build DOCKER_REPO + ":$BUILD_NUMBER"
	           }
	        }
	      }
	    }
	    stage('Push Image') {
      steps{
        script {
		   docker.withRegistry( DOCKER_REG, DOCKER_REG_KEY ) {
		            dockerImage.push()
		          }
		        }
		      }
		}

		stage('Deploy Image to K8s') {
      steps{
        script {
          sh 'cd ../'
        	sh "sed -i 's/{version}/" + BUILD_NUMBER + "/g' deployment.yaml"
	   		sh 'kubectl apply -f deployment.yaml'
		      }
		}
		}


		stage('Remove Unused docker image') {
      steps{
        sh "docker rmi $DOCKER_REPO:$BUILD_NUMBER"
      }
    }
   }


}
