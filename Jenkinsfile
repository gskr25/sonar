pipeline {
    agent any

    environment {
        function_name = 'java-sample'
    }

    stages {

        // CI Start
        stage('Build') {
            steps {
                echo 'Build'
                sh 'mvn package'
            }
        }


         stage("SonarQube analysis") {
             agent any

             when {
                 anyOf {
                     branch 'feature/*'
                     branch 'main'
                 }
             }
             steps {
                 withSonarQubeEnv('sonar') {
                      sh 'mvn clean package sonar:sonar'
                 }
             }
         }

         stage("Quality Gate") {
             steps {
                 script {
                     try {
                      timeout(time: 10, unit: 'MINUTES') {
                             waitForQualityGate abortPipeline: true
                         }
                     }
                     catch (Exception ex) {

                     }
                 }
             }
         }

        stage('Push') {
            steps {
                echo 'Push'

                sh "aws s3 cp target/sample-1.0.3.jar s3://sonar25"
            }
        }

        // Ci Ended

        // CD Started

        stage('Deployments') {
            parallel {

                stage('Deploy to Dev') {
                    steps {
                        echo 'Build'

                        sh "aws lambda update-function-code --function-name $java-sample --region us-east-1 --s3-bucket sonar25 --s3-key sample-1.0.3.jar"
                    }
                }

                stage('Deploy to test ') {
                    when {
                        branch 'main'
                    }
                    steps {
                        echo 'Build'

                         sh "aws lambda update-function-code --function-name $java-sample --region us-east-1 --s3-bucket sonar25 --s3-key sample-1.0.3.jar"
                    }
                }
            }
        }

        stage('Deploy to Prod') {
            when {
                branch 'main'
            }
            steps {
               input (
                    message: 'Are we good for Prod Deployment ?'
               )
            }
        }

        stage('Release to Prod') {
            when {
                branch 'main'
            }
            steps {
                sh "aws lambda update-function-code --function-name $java-sample --region us-east-1 --s3-bucket sonar25 --s3-key sample-1.0.3.jar"
            }
        }


        

        // CD Ended
    }

    post {
        always {
            echo "${env.BUILD_ID}"
            echo "${BRANCH_NAME}"
            echo "${BUILD_NUMBER}"

        }

        failure {
            echo 'failed'
        }
        aborted {
            echo 'aborted'
        }
    }
}
