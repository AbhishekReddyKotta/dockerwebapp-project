pipeline {
    agent {
        node {
            label "prod"
        }
    }
     tools {
        maven 'ark-maven'
    }
    stages {
        stage('code') {
            steps {
                // Get some code from a GitHub repository
                git 'https://github.com/AbhishekReddyKotta/dockerwebapp-project.git'
            }
        }
        stage('Build') {
            tools {
                    jdk 'Java-8'
            }
            steps {
                // Run the build. You must have Maven installed.
                sh 'mvn clean install'
            }
        }
        stage('CQA') {
            // tools {
            //         jdk 'Java-21'
            // }
            environment {
                // Adjust this path if your Java 21 location differs on the agent
                JAVA_HOME = '/usr/lib/jvm/java-21-amazon-corretto'
                PATH = "${JAVA_HOME}/bin:${env.PATH}"
            }
            steps {
                withSonarQubeEnv('Ark-SonarQube') {
                    // sh "mvn clean verify sonar:sonar -Dsonar.projectKey=ark-sonar"
                    sh "mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=ark-sona"

                }
            }
        }
        stage('Quality Gates') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar'
                }
            }
        }
        stage ("Artifact Nexus Upload") {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'vprofile', classifier: '', file: 'target/vprofile-v2.war', type: 'war']], 
                credentialsId: 'nexus', 
                groupId: 'com.visualpathit', 
                nexusUrl: '44.223.102.57:8081/', 
                nexusVersion: 'nexus3', 
                protocol: 'http', 
                repository: "ark-repo", 
                version: 'v2_${BUILD_NUMBER}'
            }
        }
        stage ('Image Build') {
            steps {
                sh 'cp -r target Docker-app'
                sh 'docker build -t appimage:v${BUILD_NUMBER} Docker-app'
                sh 'docker build -t dbimage:v${BUILD_NUMBER} Docker-db'
            }
        }
        stage ('Image Scan') {
            steps {
                sh ' trivy image appimage:v${BUILD_NUMBER} >> appimage-trivy-report.txt'
                sh ' trivy image dbimage:v${BUILD_NUMBER} >> appimage-trivy-report.txt'
            }
        }
        stage ('Image Registry') {
            steps {
                script {
                    // This step should not normally be used in your script. Consult the inline help for details.
                    withDockerRegistry(credentialsId: 'dockerhub') {
                        // some block
                        sh 'docker tag appimage:v${BUILD_NUMBER} arkotta27/dockerprojectapp:v${BUILD_NUMBER}'
                        sh 'docker tag dbimage:v${BUILD_NUMBER} arkotta27/dockerprojectdb:v${BUILD_NUMBER}'
                        sh 'docker push arkotta27/dockerprojectapp:v${BUILD_NUMBER}'
                        sh 'docker push arkotta27/dockerprojectdb:v${BUILD_NUMBER}'
                    }
                }
            }
        }
        stage ('Deploy') {
            steps {
                sh 'docker stack deploy myapp --compose-file=compose.yml'
            }
        }
    }
}
