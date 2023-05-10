pipeline {
    agent any
    environment {
        SERVER_IP = sh(script: 'curl -s http://169.254.169.254/latest/meta-data/public-ipv4', returnStdout: true).trim()
    }
    tools {
        maven 'my_mvn'
    }
    stages {
        stage("Checkout") {   
            steps {               	 
                git branch: 'main', url: 'https://github.com/manikcloud/manik-calculator.git'        	 
            }    
        }
        stage('Maven Clean') {
            steps {
                sh "mvn clean"  	 
            }
        }
        stage('Maven Build') {
            steps {
                sh "mvn compile"  	 
            }
        }
        stage("Unit Test") {          	 
            steps {  	 
                sh "mvn test"          	 
            }
        }
        stage("SonarQube Analysis") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'sonarqube', passwordVariable: 'password', usernameVariable: 'username')]) {
                    withSonarQubeEnv('sonarqube-server') {
                        sh "mvn verify sonar:sonar -Dsonar.host.url=http://${SERVER_IP}:9000 -Dsonar.login=${username} -Dsonar.password=${password}"
                    }
                }
            }
        }
        stage("Maven Package") {
            steps {
                sh "mvn package" 
            }
        }
        stage("Deploy On Server") {          	 
            steps {  	 
                deploy adapters: [tomcat9(credentialsId: 'tomcat-9', path: '', url: "http://${SERVER_IP}:8090")], contextPath: '/manik-calculator', war: '**/target/*.war'         	 
            }
        }  	
    }
    post {
        always {
            junit 'target/surefire-reports/*.xml'
        }
        success {
            echo "App URL: http://${SERVER_IP}:8090/manik-calculator/"
            emailext (
                to: 'varunmanik1@gmail.com',
                subject: "SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: "The job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' completed successfully."
            )
        }
        failure {
            echo "Failed to deploy application to http://${SERVER_IP}:8090/manik-calculator/"
            emailext (
                to: 'varunmanik1@gmail.com',
                subject: "FAILURE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: "The job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' failed."
            )
        } 
    }
}
