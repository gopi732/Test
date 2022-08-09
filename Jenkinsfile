pipeline {
  agent {
	label 'slave'
  }
  stages { 
      stage('CleanUp WorkSpace & Git Checkout') {
          steps {
              // Clean before build
              cleanWs()
              // We need to explicitly checkout from SCM here
              checkout scm
          }
      }	  
      stage('Build & Test') {
         agent { 
		docker { 
		  image 'python:3.8' 
   	          reuseNode true
	        }	
	 }
	 environment {   
		http_proxy = 'http://127.0.0.1:3128/'
		https_proxy = 'http://127.0.0.1:3128/'
		ftp_proxy = 'http://127.0.0.1:3128/'
		socks_proxy = 'socks://127.0.0.1:3128/'     
	 }		
	 steps {
	      echo "Building Image and Conatiner"
	      sh 'pip install virtualenv'
	      sh 'pip install -r requirements.txt'
	      sh 'pip install -e .'
	      sh 'python3 -m pytest'
	 }
      }	 
      stage('Code Analysis') {
	  environment {
	       scannerHome = tool 'SonarQube Scanner'
	  }
	  steps {
	      withSonarQubeEnv('admin') {
		   sh '${scannerHome}/bin/sonar-scanner \
		      -D sonar.projectKey=CI-Test \
		      -D sonar.python.coverage.reportPaths=*.js,*.html,*.json,*.css'
	      }
          }
      }
      stage('Deploy Atrifacts') {
	  steps {
	      rtUpload (
		 serverId: 'JFrog',
		 spec: '''{
 			"files" :[
			  {
		            "pattern": "coverage/",
		            "target": "CI-Test/"
	         	  }
		        ]
		 }'''
	     )
	 }
      }
   }
   post {
	 always {
	     echo 'I have finished'
	 }
	 success { 
	     echo 'I succeeded!'
	 }
	 unstable {
	     echo 'I am unstable :/'
	 }
	 failure {
             echo 'I failed :('
	 }
	  changed {
            echo 'Things are different...'
        }
   }	  
}

