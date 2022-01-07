# Jenkinspipeline

pipeline {
environment {
registry = "d2ssharma/d2ssharma"
registryCredential = 'd2ssharma'
dockerImage = ''
}
   agent any
   stages {
    stage('Checkout') {
      steps {
        script {
           git credentialsId: 'jenkins-user-github', url: 'https://github.com/spring-projects/spring-petclinic.git'
           sh "git checkout master"
           dir('spring-petclinic') {
            sh "pwd"
            dockerImage = docker.build registry + ":$BUILD_NUMBER"
            }
          }
       }
    }
	stage('Deploy our image') {
	  steps {
	      script {
			docker.withRegistry( '', registryCredential ) {
			dockerImage.push()
			}
	      }
		}
	}
    stage('Upload') {
    steps {
        script {
        dir('spring-petclinic'){
        pwd(); 
        withAWS(region:'ap-south-1',credentials:'2345399d-5bfd-4cca-bda2-95f62d976811') {
        def identity=awsIdentity();
        s3Upload(bucket:"pipetest12", includePathPattern:'**/*');
        }
        }
        };
    }
    }
  }
}
