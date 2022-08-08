pipeline {
    agent any
    environment{   
        AWS_S3_BUCKET = "amjad-belt2d2-artifacts-123456"
        ARTIFACT_NAME = "hello-world.jar"
        AWS_ACCESS_KEY_ID     = credentials('amjad-aws-secret-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('amjad-aws-secret-access-key')
        AWS_EB_APP_NAME = "amjad-jenkins"
        AWS_EB_APP_VERSION = "${BUILD_ID}"
        AWS_EB_ENVIRONMENT = "Amjadjenkins-env"
    }


    stages {
        
        stage('quality scan'){
            steps{
                sh '''
            sudo docker run -d --name sonarqube -p 80:9000 -v sonarqube_data:/opt/sonarqube/data -v sonarqube_extensions:/opt/sonarqube/extensions -v sonarqube_logs:/opt/sonarqube/logs sonarqube

                '''
            }
        }
 
        stage('Validate') {
            steps {
                sh "mvn validate"
                sh "mvn clean"
            }
        }

        stage('Build') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Test') {
            steps {
                sh "mvn test"
                
            }
       
            post {
                always{
                    junit '**/target/surefire-reports/TEST-*.xml'
                }
            }
        }


        stage('Package') {
            steps {
                sh "mvn package"
                
            }
            post{
                success{
                    archiveArtifacts artifacts: '**/target/**.jar', followSymlinks: false
                }
            }
        }
    
        stage('publish artfacts to s3') {
            steps {
                sh "aws configure set region us-east-1" 
                sh "aws s3 cp ./target/**.jar s3://$AWS_S3_BUCKET/$ARTIFACT_NAME"
            }
        }

        stage('Deploy') {
            steps {

                sh 'aws elasticbeanstalk create-application-version --application-name $AWS_EB_APP_NAME --version-label $AWS_EB_APP_VERSION --source-bundle S3Bucket=$AWS_S3_BUCKET,S3Key=$ARTIFACT_NAME'

                sh 'aws elasticbeanstalk update-environment --application-name $AWS_EB_APP_NAME --environment-name $AWS_EB_ENVIRONMENT --version-label $AWS_EB_APP_VERSION'
            }
        }

    }
}
