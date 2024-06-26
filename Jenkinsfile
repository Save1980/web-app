COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]
pipeline {
    agent any
    tools {
        maven "Maven 3.9.6"
    }
    stages {
        stage("Git clone") {
            steps {
                git branch: 'main', url: 'https://github.com/Save1980/web-app.git'
            }
        }

        stage("Build with maven") {
            steps {
                sh "mvn clean"
            }
        }

        stage("Testing with Maven") {
            steps {
                sh "mvn test"
            }
        }

        stage("Packaging with Maven") {
            steps {
                sh "mvn package"
            }
        }

        stage("SONARQUBE ANALYSIS") {
            environment {
                ScannerHome = tool 'sonar5.0'
            }
            steps {
                script {
                    withSonarQubeEnv('sonarqube') {
                        sh "${ScannerHome}/bin/sonar-scanner -Dsonar.projectKey=Jomacs"
                    }
                }
            }
        }
        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
        }

        stage("Upload to NEXUS") {
            steps {
                nexusArtifactUploader(
                    artifacts: [
                        [artifactId: 'maven-web-application', classifier: '', file: "/var/lib/jenkins/workspace/first-pipeline-job-in-jenkins/target/web-app.war", type: 'war']
                    ],
                    credentialsId: 'nexus-id2',
                    groupId: 'com.mt',
                    nexusUrl: '16.16.57.8:8081',
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    repository: 'webapp-snapshot',
                    version: '3.1.2-SNAPSHOT'
                )
            }
        }
        stage("Deploy to Tomcat"){
            steps{
                deploy adapters: [tomcat9(credentialsId: 'tomcat-credential2', path: '', url: 'http://16.171.12.103:8080')], contextPath: null, war: 'target/*.war'
            }
        }
    }
     post {
        always {
            slackSend channel: 'team-europe', 
                      color: COLOR_MAP[currentBuild.currentResult],
                      message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}"
        }
    }
  
}

