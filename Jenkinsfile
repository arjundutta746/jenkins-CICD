pipeline {
    agent any
    tools {
        jdk "OracleJDK-11"
        maven "MAVEN-3"
    }

    stages {
        stage ("Fetch Code") {
            steps {
                git branch: 'main', url: 'https://github.com/arjundutta746/vprofile-project'
            }
        }

        stage ('Build') {
            steps {
                sh 'mvn install -DskipTests'
            }

            post {
                success {
                    echo 'Archiving artifacts'
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage ('Unit Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage ('Sonar Analysis') {
            environment {
                scannerHome = tool 'sonar4.7'
            }

            steps {
                withSonarQubeEnv('sonar') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Upload artifacts to Nexus Repo"){
            steps {
                 nexusArtifactUploader(
                            nexusVersion: 'nexus3',
                            protocol: 'http',
                            nexusUrl: "172.31.35.235:8081",
                            groupId: 'QA', // Replace with your groupId
                            version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}", // Replace with your artifact version
                            repository: 'vprofile-repo',
                            credentialsId: 'NexusLogin', // You need to configure Nexus credentials in Jenkins credentials
                            artifacts: [
                                [artifactId: 'vproapp', 
                                classifier: '', 
                                file: "target/vprofile-v2.war", 
                                type: 'war']
                            ]
                        )
                
            }
        }
    }
}
