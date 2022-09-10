pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_S3_BUCKET = "teamcity-proj"
        ARTIFACT_NAME = "spring-boot.jar"
        AWS_EB_APP_NAME = "teamcity-proj"
        AWS_EB_APP_VERSION = "${BUILD_ID}"
        AWS_EB_ENVIRONMENT = "Teamcityproj-env"
    }
    stages {
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
            post{
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
                    archiveArtifacts artifacts: 'target/Maven-java-spring-boot-0.0.1-SNAPSHOT.jar', followSymlinks: false
                }
            }
        }
        stage('Publish artifacts to S3 Bucket') {
            steps {
                sh "aws configure set region us-west-1"
                sh "aws s3 cp ./target/Maven-java-spring-boot-0.0.1-SNAPSHOT.jar s3://$AWS_S3_BUCKET/$ARTIFACT_NAME"
            }
         }
        stage('Deploy') {
            steps {
                sh 'aws elasticbeanstalk create-application-version --application-name teamcity --version-label $AWS_EB_APP_VERSION --source-bundle S3Bucket=teamcity-5,S3Key=te.jar'
                sh 'aws elasticbeanstalk update-environment --application-name teamcity --environment-name Teamcity-env --version-label 1'
            }
         }
        

    }
}