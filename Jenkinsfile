def scan_type
 def target
 pipeline {
    agent any
	parameters {
         	choice  choices: ["Baseline", "APIS", "Full"],
                 	description: 'Type of scan that is going to perform inside the container',
                 	name: 'SCAN_TYPE'
		booleanParam defaultValue: true,
                 	description: 'Parameter to know if wanna generate report.',
                 	name: 'GENERATE_REPORT'
     	}
	tools {
		maven 'Maven'
	}
	
	environment {
		PROJECT_ID = 'tech-rnd-project'
                CLUSTER_NAME = 'jenkins-jen-cluster'
                LOCATION = 'asia-south1-a'
                CREDENTIALS_ID = 'kubernetes'	
		TAR = ''
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
			sh '''
			    echo "Pulling up last OWASP ZAP container --> Start"
			    docker pull owasp/zap2docker-stable
			    
			    echo "Starting container --> Start"
			    docker run -dt --name owasp \
    			    owasp/zap2docker-stable \
    			    /bin/bash
			    
			    
			    echo "Creating Workspace Inside Docker"
			    docker exec owasp \
    			    mkdir /zap/wrk
			'''
			    }
		    }
	    stage('Scanning target on owasp container') {
             steps {
                 script {
		     sh 'sleep 10'
			sh 'gcloud container clusters get-credentials jenkins-jen-cluster --zone asia-south1-a --project tech-rnd-project'
			sh 'kubectl get pods'	
			sh 'kubectl get service myapp > intake.txt'
                        sh '''
			    
			    awk '{print $4}' intake.txt > extract.txt
			    grep -Eo "(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)" extract.txt > finalout.txt
			    
			    
			'''
			 sh """
			    ip=\$(cat finalout.txt)
			    host="http://\${ip}"
			    params.TARGET = $host
		        """
                     scan_type = "${params.SCAN_TYPE}"
                     echo "----> scan_type: $scan_type"
		     target = "${params.TARGET}"
			 
                     if(scan_type == "Baseline"){
                         sh """
                             docker exec owasp \
                             zap-baseline.py \
                             -t $target \
                             -r report.html \
                             -I
                         """
                     }
                     else if(scan_type == "APIS"){
                         sh """
                             docker exec owasp \
                             zap-api-scan.py \
                             -t $target \
                             -r report.html \
                             -I
                         """
                     }
                     else if(scan_type == "Full"){
                         sh """
                             docker exec owasp \
                             zap-full-scan.py \
                             -t $target \
                             //-x report.html
                             -I
                         """
                         //-x report-$(date +%d-%b-%Y).xml
                     }
                     else{
                         echo "Something went wrong..."
                     }
				    
		}
		}
         }
    }
}
