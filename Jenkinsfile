
def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]




pipeline {
    agent any
    
    
    environment {
      WORKSPACE = "${env.WORKSPACE}"
      
    }
    
    tools {
        jdk 'localJdk'
        maven 'localMaven'
    }


    stages {
        stage('Git Checkout') {
            steps {
                echo 'cloning git'
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/eniornella/jenkins-app-pipeline-project.git']]])
            }
        }
        
         stage('Build') {
            steps {
                sh 'mvn clean package'
                sh 'java -version'
            }
            
         
            post { 
                success { 
                    echo 'archiving'
                    archiveArtifacts artifacts: '**/*.war',followSymlinks: false
                }
            } 
            
            
        }
        
        stage('Unit Test') {
            steps {
                sh 'mvn test'
              
            } 
            
        }
            stage('Integration Test'){
                steps {
                    sh 'mvn verify -DskipUnitTests'
                }
            }
              stage ('Checkstyle Code Analysis'){
                  steps {
                      sh 'mvn checkstyle:checkstyle'
            }
            
            post {
               success {
                   echo 'Generated Analysis Result'
                }
            }
      
        }
      
            stage('SonarQube Scan') {
                steps {
                    
                    withSonarQubeEnv('SonarQube'){
                    
                    sh """mvn sonar:sonar \
                   -Dsonar.projectKey=JavaWebApp \
                   -Dsonar.host.url=http://172.31.19.18:9000 \
                   -Dsonar.login=b9773512fcab16610100f94fadebe8b839dfd77e"""
                } 
            
            }
             
            }
            
             stage("Quality Gate"){
                 steps{
                     
                    waitForQualityGate abortPipeline: true
                   
                }

            }
            
             stage('Upload to Artifactory') {
                steps {
                    sh "mvn clean deploy -DskipTests"
                }
            }
            
            stage('Deploy to DEV') {
                environment {
                     HOSTS = "dev"
            }
                steps {
                    sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
        
                }
            
            }
        
             stage('Approval') {
                steps {
                    input('Do you want to proceed?')
                }
            }
            
             stage('Deploy to PROD') {
                environment {
                    HOSTS = "prod"
                }
                steps {
                     sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
                    }
                }
             
            }
             post {
                 always {
                 echo 'Slack Notifications.'
                 slackSend channel: 'glorious-w-f-devops-alerts',
                 color: COLOR_MAP[currentBuild.currentResult],
                 message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
                }
            }    
        }
        
    



