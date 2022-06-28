pipeline {
    agent any
	tools {
		maven 'Maven'
	}
	
	environment {
		PROJECT_ID = 'tech-rnd-project'
                CLUSTER_NAME = 'jen-cluster'
                LOCATION = 'asia-south1-a'
                CREDENTIALS_ID = 'kubernetes'		
	}
	
    stages {
	    stage('Scm Checkout') {
		    steps {
			    checkout scm
		    }
	    }
	    
	    stage('Build') {
		    steps {
			    sh 'mvn clean package'
		    }
	    }
	    
	    stage('Test') {
		    steps {
			    echo "Testing..."
			    sh 'mvn test'
		    }
	    }
	    
	    stage('Build Docker Image') {
		    steps {
			    sh 'whoami'
			    script {
				    myimage = docker.build("fazilniveus/devops:${env.BUILD_ID}")
			    }
		    }
	    }
	    
	    stage("Push Docker Image") {
		    steps {
			    script {
				    echo "Push Docker Image"
				    withCredentials([string(credentialsId: 'dockerhub', variable: 'dockerhub')]) {
            				sh "docker login -u fazilniveus -p ${dockerhub}"
				    }
				        myimage.push("${env.BUILD_ID}")
				    
			    }
		    }
	    }
	    
	    stage('Deploy to K8s') {
		    steps{
			    echo "Deployment started ..."
			    sh 'ls -ltr'
			    sh 'pwd'
				sh "sed -i 's/tagversion/${env.BUILD_ID}/g' deployment.yaml"
				echo "Start deployment of deployment.yaml"
				step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'deployment.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
			    echo "Deployment Finished ..."
		    }
	    }
	    stage('Zap Installation') {
                    steps {
                        sh 'echo "Hello World"'
                        sh '''
                            sudo docker run --detach --name zap -u zap -v "$(pwd)/reports":/zap/reports/:rw   -i owasp/zap2docker-stable zap.sh -daemon -host 0.0.0.0 -port 8080   -config api.addrs.addr.name=.* -config api.addrs.addr.regex=true   -config api.disablekey=true
                            sudo docker exec zap zap-cli --verbose quick-scan https://niveussolutions.com
		            sudo docker exec zap zap-cli --verbose report -o /zap/owasp-quick-scan-report.html --output-format html
                        '''
                    }
            }
    }
}
