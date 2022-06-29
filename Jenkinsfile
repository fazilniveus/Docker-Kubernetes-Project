pipeline {
    agent any
	tools {
		maven 'Maven'
	}
	
	environment {
		PROJECT_ID = 'tech-rnd-project'
                CLUSTER_NAME = 'jenkins-jen-cluster'
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
			    sh 'mvn clean'
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
			    sh '''

			    '''
			    
		    }
	    }
	    stage('Zap Installation') {
                    steps {
                        sh 'echo "Hello World"'
			sh 'sleep 60'
			sh 'gcloud container clusters get-credentials jenkins-jen-cluster --zone asia-south1-a --project tech-rnd-project'
			sh 'kubectl get pods'	
			sh 'sleep 10'
			sh 'kubectl get service myapp > intake.txt'
                        sh '''
			    
			    awk '{print $4}' intake.txt > extract.txt
			    grep -Eo "(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)" extract.txt > finalout.txt
			    ip=$(cat finalout.txt)			    
			    host="http://${ip}"
			    
                            
 			    
			    docker run -d --name=zap owasp/zap2docker-stable zap-cli quick-scan --self-contained     --start-options '-config api.disablekey=true' $host tail -f /dev/null
			    4a4f1abb49e1
			    docker exec -d zap zap-cli --verbose report -o /zap/owasp-quick-scan-report.html --output-format html
			    sudo docker cp zap:/zap/owasp-quick-scan-report.html /home/mohammad_fazil/ 
		            
			    
                        '''
                    }
            }
    }
}
