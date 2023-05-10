pipeline {
    agent any
    environment {
        registry = "mohamedmohsen38/vprofile-app"
        registryCredential = "dockerhub"
       

    }
    stages{

        stage('Build artifact'){
            steps {
                sh 'mvn clean install -DskipTests'

            }

            post {
                success {
                    echo "now archiving artifact"
                    archiveArtifacts artifacts: '**/target/*.war'
                }

            }
        }

        stage('Unit Tests'){
            steps {
                sh "sed -i 's/0.7.2.201409121644/0.8.5/g' pom.xml"
                sh 'mvn test'

            }
        }

        stage('code analysis with checkstyle'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success{
                    echo "generated code analysis results"

                }
            }
        }


#        stage('code analysis with sonarqube'){
#            steps{
#                withSonarQubeEnv('sonar-pro'){
#                     sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
#                   -Dsonar.projectName=vprofile-repo \
#                   -Dsonar.projectVersion=1.0 \
#                   -Dsonar.sources=src/ \
#                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
#                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
#                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
#                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
#               }
#           }
#        }

        stage('build App image') {
            steps {
                script {
                    dockerImage = docker.build registry + ":V$BUILD_NUMBER"
                }

            }

        }

        stage ('upload Image') {
            steps {
                script {
                    docker.withRegistry('', registryCredential)
                    dockerImage.push("V$BUILD_NUMBER")
                    dockerImage.push('latest')
                }
            }
        }
        
        stage ('remove unused docker images'){
            steps {
                sh "docker rmi $registry:V$BUILD_NUMBER"
            }
        }

        stage ('kubenetes deploy') {
            agent {label 'jenkins-slave'}
            steps {
                  sh "helm upgrade --install --force vproifle-stack helm/vprofilecharts --set appimage=${registry}:${BUILD_NUMBER} --namespace prod"
            }
        }


    }
    
}
