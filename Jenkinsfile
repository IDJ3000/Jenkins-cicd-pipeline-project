def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]

def currentDate = new Date()
def formattedDate
def dateFormat = new java.text.SimpleDateFormat("yyyy-MM-dd HH:mm:ss")
dateFormat.setTimeZone(TimeZone.getTimeZone("America/Chicago"))

pipeline {
    agent any
    
    tools {
        maven 'localMaven'
        jdk 'localJDK'
    }
    
    environment{
          WORKSPACE = "${env.WORKSPACE}"
      }

    stages {
        stage('Stage 1: Greeting') {
            steps {
                echo '################################################'
                echo 'USING MAVEN FOR BUILDING'
                echo '################################################'
            }
        }
        
        stage('Git Checkout') {
            steps {
                echo 'Hello World'
                git branch: 'main', credentialsId: 'github-login', url: 'https://github.com/IDJ3000/Jenkins-cicd-pipeline-project.git'
            }
        }
        
        
       stage('Build') {
            steps {
                sh 'mvn clean package'
            }
            
        
        post { 
            success { 
            archiveArtifacts artifacts: '**/*.war', followSymlinks: false
                }
            }
        }
        
        stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('Integration Test') {
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }
        
        stage('Checkstyle Code Analysis') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }
        
        stage('SonarQube scan') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                    mvn sonar:sonar \\
                          -Dsonar.projectKey=java-web-app \\
                          -Dsonar.host.url=http://172.31.80.116:9000 \\
                          -Dsonar.login=093904c8e68bb40938a7728830c36dd2c849d5b5
                          '''
                }
            }
        }
        
         stage('Quality Gate') {
            steps {
                waitForQualityGate true
            }
        }
        
        stage('Upload Artifact to Nexus') {
            steps {
                sh 'mvn deploy -DskipUnitTests'
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
        
    }
    
     post {
            always {
                echo 'Slack Notifications.'
                slackSend channel: '#jjtech-champions-app-pipeline', //update and provide your channel name
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL} \n ################################## \n BUILDING WITH MAVEN By Engr Precious \n ##################################"
            }
          }
}