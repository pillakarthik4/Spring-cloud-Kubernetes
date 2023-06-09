@Library('jenkins-shared-library@main') _
pipeline {

  agent any
  
  parameters {
	choice(name: 'action', choices: 'create\nrollback', description: 'Create/rollback of the deployment')
    string(name: 'ImageName', description: "Name of the docker build", defaultValue: "Spring-cloud-Kubernetes")
	string(name: 'ImageTag', description: "Name of the docker build",defaultValue: "v1")
	string(name: 'AppName', description: "Name of the Application",defaultValue: "Spring-cloud-Kubernetes")
    string(name: 'docker_repo', description: "Name of docker repository",defaultValue: "karthikpilla")
  }
      
  tools{ 
        maven 'maven3'
    }
    stages {
        stage('Git-Checkout') {
            when {
				expression { params.action == 'create' }
			}
            steps {
                gitCheckout(
                    branch: "main",
                    url: "https://github.com/pillakarthik4/Spring-cloud-Kubernetes.git"
                )
            }
        }
        stage('Build-Maven'){
            when {
				expression { params.action == 'create' }
			}
    		steps {
        		dir("${params.AppName}") {
        			sh 'mvn clean package'
        		}
    		}
	    }
	    stage("DockerBuild and Push") {
	        when {
				expression { params.action == 'create' }
			}
	        steps {
	            dir("${params.AppName}") {
	                dockerBuild ( "${params.ImageName}", "${params.docker_repo}" )
	            }
	        }
	    }
	    stage("Docker-CleanUP") {
	        when {
				expression { params.action == 'create' }
			}
	        steps {
	            dockerCleanup ( "${params.ImageName}", "${params.docker_repo}" )
			}
		}
			    stage("Ansible Setup") {
	        when {
				expression { params.action == 'create' }
			}
	        steps {
	            sh 'ansible-playbook ${WORKSPACE}/Spring-cloud-Kubernetes/service.yml'
			}
		}
	    stage("Create deployment") {
			when {
				expression { params.action == 'create' }
			}
	        steps {
	            sh 'echo ${WORKSPACE}'
	            sh 'kubectl create -f ${WORKSPACE}/Spring-cloud-Kubernetes/kubernetes-configmap.yml'
	        }
	    }
	    stage ("wait_for_pods"){
	    steps{
              
                sh 'sleep 300'
             
	    }
	    }
		stage("rollback deployment") {
	        steps {	            	         	           
	              sh """
	                   kubectl delete deploy ${params.AppName}
					    kubectl delete svc ${params.AppName}
				  """	       
	        }
	    }
    }
}
