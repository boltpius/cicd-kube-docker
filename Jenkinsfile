pipeline {

    agent any
/*
	tools {
        maven "maven3"
    }
*/
    environment {
        registry = "boltpius/vproappdock"
        registryCredential = 'dockerhub' 
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
                scannerHome = tool 'sonarscanner'
            }

            steps {
                withSonarQubeEnv('sonarserver') {
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

        stage("Build Docker Image") {
            steps{
              script {
                dockerImage = docker.build registry + ":v$BUILD_NUMBER" // this build the docker image and give it a name and a tag.
               
            }
        }
    }
       
        stage("Upload image") {
            steps{
                script {
                    docker.withRegistry('', registryCredential ) {
                        dockerImage.push("v$BUILD_NUMBER")// these are docker functions use to push docker image to dockerhub and the 'dockerimage' is where we built the image from the last stage so we reference it in this stage to push.
                         dockerImage.push('latest')
                    }
                }
            }
        }

        stage('Remove Unused docker image') {
          steps{
            sh "docker rmi $registry:v$BUILD_NUMBER" // this stage basically removes unused docker images so it doesnt use up al the space in the jenkins server.

          }
        }

        stage('Kubernetes Deploy') {
	      agent { label 'kopsnode' }
            steps {
                    sh "helm upgrade --install --force vproifle-stack helm/vprofilecharts --set appimage=${registry}:v${BUILD_NUMBER} --namespace prod"
            }
        }
        
    


}

}
