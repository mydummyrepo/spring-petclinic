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
        jfrog 'jfrog-cli'
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
                    sh '''
                            mvn clean package sonar:sonar \
                            -Dsonar.organization=mydummyrepo \
                            -Dsonar.projectKey=mydummyrepo_spring-petclinic \
                            -Dsonar.host.url=https://sonarcloud.io \
                            -Dsonar.login=$SONAR_TOKEN
                        '''
                }
                junit testResults: '**/surefire-reports/*.xml'
                archive '**/target/spring-petclinic-*.jar'
            }
        }
        stage('Exec Maven commands') {
            steps {
                // Configure Maven project's repositories
                    jf 'mvn-config --repo-resolve-releases ltdev-libs-release --repo-resolve-snapshots ltdev-libs-snapshot --repo-deploy-releases ltdev-libs-release-local --repo-deploy-snapshots ltdev-libs-snapshot-local'

                    // Install and publish project
                    jf 'mvn clean install'
            }
        }

        stage('Publish build info') {
            steps {
                jf 'rt build-publish'
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
