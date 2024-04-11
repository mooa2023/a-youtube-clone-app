pipeline {
    agent any 
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    
    environment {
	    SCANNER_HOME = tool 'sonar-scanner'
    }
    
    stages{
        stage("Cleanup Workspace") {
                steps {
                cleanWs()
                }
        }

        stage("Checkout from Git") {
                steps {
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/mooa2023/a-youtube-clone-app.git'
                }
        }

        stage("SonarQube Analysis"){
           steps {
		        withSonarQubeEnv('SonarQube-Server') { 
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Youtube-CICD \
                    -Dsonar.projectKey=Youtube-CICD'''
	           }	
           }
        }

        stage("Quality Gate"){
           steps {
               script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'SonarQube-Token'
                }	
            }
        }

        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }

        stage("Trivy FS Scan") {
           steps {
	            sh "trivy fs . > trivyfs.txt"
           }
        }
    
        stage("Docker Build & Push") {
            steps{
                script{
                    withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker'){
                      sh "docker build -t youtube-clone4 ."
                      sh "docker tag youtube-clone mooa2023/youtube-clone4:latest "
                      sh "docker push mooa2023/youtube-clone4:latest "
                    }
                }
            }    
        }
    
        stage("TRIVY Image Scan") {
            steps{
                sh "trivy image mooa2023/youtube-clone4:latest > trivyimage.txt"
            }
        }
        
        stage('Deploy to Kubernets'){
            steps{
                script{
                    dir('Kubernetes') {
                      withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'kubernetes', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                      sh 'kubectl delete --all pods'
                      sh 'kubectl apply -f deployment.yml'
                      sh 'kubectl apply -f service.yml'
                      }   
                    }
                }
            }
        }
        
    }
    
    post {
        always {
            emailext attachLog: true,
                subject: "'${currentBuild.result}'",
                body: "Project: ${env.JOB_NAME}<br/>" +
                    "Build Number: ${env.BUILD_NUMBER}<br/>" +
                    "URL: ${env.BUILD_URL}<br/>",
                to: 'maryogundiran24@gmail.com',
                attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
        }
    }
}
