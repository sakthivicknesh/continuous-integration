pipeline{
    agent any
    tools{
        maven "MAVEN3.9"
        jdk "JDK17"
    }

    stages{

        stage('Fetch the source code'){
            steps{
                git branch: 'atom' ,url : 'https://github.com/hkhcoder/vprofile-project.git'         
            }    
        }

        stage('Build'){
            
            steps{
                sh "mvn install -DskipTests"        
            } 
            post{

                success{
                    archiveArtifacts artifacts: '**/*.war'
                }

            }   
        }


        stage('unit test'){
            steps{
                sh 'mvn test'         
            }    
        }

        stage('Checkstyle Analysis'){
            steps{
                sh 'mvn checkstyle:checkstyle'         
            }    
        }


        stage('CODE ANALYSIS with SONARQUBE') {
            environment {
                scannerHome = tool 'sonar8.1.0.6389'
            }

            steps {
                withSonarQubeEnv('sonarserver') {
                    sh '''
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=vprofile \
                        -Dsonar.projectName=vprofile \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src/  \
                        -Dsonar.java.binaries=target/classes \
                        -Dsonar.junit.reportPaths=target/surefire-reports \
                        -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml \
                        -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        Stage('Upload to Nexus'){
            steps{
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: '172.31.39.217:8081',
                    groupId: 'QA',
                    version: '${BUILD_ID}-${BUILD_TIMESTAMP}',
                    repository: 'vprofile-repo',
                    credentialsId: 'nexus',
                    artifacts: [
                        [artifactId: 'vprofile', 
                        classifier: '', 
                        file: 'target/vprofile-v2.war', 
                        type: 'war']
                    ]
                )
            }
        }


    }
}