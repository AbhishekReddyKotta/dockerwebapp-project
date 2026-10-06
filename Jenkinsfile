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
    }
}
