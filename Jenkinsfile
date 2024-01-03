pipeline {
	agent any
    
    environment {
        AWS_CREDENTIALS = credentials('579464346048')
        CODEBUILD_PROJECT = 'jenkins-prueba' //CodeBuild project name

        S3_BUCKET = 'chapinbet-smartplay-build'
    }
    
    stages{
    
    	stage('Copy to S3'){
            steps {
                script {
                    //Copia el código al buck S3 y borrar contenido local
                    sh "aws s3 cp /var/lib/jenkins/workspace/github_prueba_main s3://$S3_BUCKET/prueba_git/ --recursive"
                    sh 'rm -rf /var/lib/jenkins/workspace/github_prueba_main/*'
                }
            }
        }

        stage('Start CodeBuild compilation'){
            steps {
                script {
                    //Inicia el proyecto de codebuild
                    sh 'aws codebuild start-build --project-name $CODEBUILD_PROJECT'
                }
            }
        }

        stage('Deploy to CodeDeploy') {
            
        }
    }
    
}