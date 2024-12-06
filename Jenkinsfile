pipeline {
    agent{label 'java'}
    options{
        timeout(time: 1, unit: 'HOURS')
    }
    triggers {
        cron('H * * * 1-5')
    }
    tools{
        maven 'apache-maven-3.9.9'
        jdk 'openjdk-17-jdk'
    }
    environment {
        SONAR_TOKEN = credentials('SONAR_TOKEN') // Secure SonarQube token
    }
    stages {
        stage('SCM'){
            steps{
                git(
                    url: 'https://github.com/mydummyrepo/spring-petclinic.git',
                    branch: 'main'
                )
            }
        }
        stage('Build and SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarcloud'){
                    sh """
                            mvn clean package sonar:sonar \
                            -Dsonar.organization=mydummyrepo \
                            -Dsonar.projectKey=mydummyrepo_spring-petclinic \
                            -Dsonar.host.url=https://sonarcloud.io \
                            -Dsonar.login=$SONAR_TOKEN
                        """
                }
            }
        }
        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // If the Quality Gate fails, abort the pipeline
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
