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
			'''
			    
			    sh 'echo "Pulling up last OWASP ZAP container --> Start"'
			    sh 'docker pull owasp/zap2docker-stable'

			    echo "Starting container --> Start"
			    sh """
    				docker run -dt --name owasp \
    				owasp/zap2docker-stable \
    				/bin/bash
			    """

			    sh 'echo "Creating Workspace Inside Docker"'
			    sh """
    				docker exec owasp \
    				mkdir /zap/wrk
			    """

			    sh 'echo "Starts Baseline Scan"'
			    sh """
    				docker exec owasp \
    				zap-baseline.py \
    				-t $host \
    				-r report.html \
    				-I
			    """

			    sh 'echo "Starts API Scan"'
			    sh """
    				docker exec owasp \
    				zap-api-scan.py \
    				-t $host \
    				-r report.html \
    				-I
			    """

			    echo "Starts FULL Scan"
			    sh """
    				docker exec owasp \
    				zap-full-scan.py \
    				-t $host \
    				-r report.html
    				-I
			    """                    

		    	    sh 'echo "Copying Report to Workspace"'
		    	    sh '''
    				docker cp owasp:/zap/wrk/report.html ${WORKSPACE}/report.html
		    	    '''       

		    	    sh 'echo "Removing container"'
		    	    sh '''
    				docker stop owasp
    				docker rm owasp
			    '''
			    
			    
                    }
            }
    
}
