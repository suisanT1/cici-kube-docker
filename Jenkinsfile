pipeline {
    
	agent any
/*	
	tools {
        maven "maven3"
    }
*/	
    environment {
        registry = "suibiksan/vprofileapp"
        registryCredential = 'docker'
        ARTVERSION = "${env.BUILD_ID}"
    }
	
    stages{
        
        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('UNIT TEST'){
                steps {
                    sh 'mvn test'
                }
            }

        stage('INTEGRATION TEST'){
                steps {
                    sh 'mvn verify -DskipUnitTests'
                }
            }
		
        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('CODE ANALYSIS with SONARQUBE') {
          
		  environment {
             scannerHome = tool 'sonarscanner4'
          }

          steps {
            withSonarQubeEnv('sonar-pro') {
               sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
            }

            timeout(time: 10, unit: 'MINUTES') {
               waitForQualityGate abortPipeline: true
            }
          }
        }

        stage('BUILD DOCKER IMAGE') {
            steps {
                script {
                    dockerImage = docker.build registry + ":${ARTVERSION}"
                }
            }
        }
        stage('PUSH DOCKER IMAGE') {
            steps {
                script {
                    docker.withRegistry( '', registryCredential ) {
                        dockerImage.push('V${ARTVERSION}')
                        dockerImage.push('latest')
                    }
                }
            }
        }
        stage ('Remove Unused Images') {
            steps {
                sh 'docker rmi $registry:V${ARTVERSION}'
            }
        }
        stage ('Deploy to Kubernetes') {
         agent { label 'KOPS'}
           steps {
            sh 'helm upgrade --install --force vprofile-stack helm/vprofilechart --set appimage=${registry}:V${ARTVERSION} --set appversion=${ARTVERSION}'
           }
        }
    }
}