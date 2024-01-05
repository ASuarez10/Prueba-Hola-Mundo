pipeline {
	agent any
    
    environment {
        //Credenciales
        //AWS_CREDENTIALS_ST = credentials('579464346048') //Credenciales AWS Security Tooling
        //AWS_CREDENTIALS_CB = credentials('chapinbet_credentials') //Credenciales AWS ChapinBet

        //Constantes info
        CODEBUILD_PROJECT = 'jenkins-prueba' //CodeBuild project name
        S3_BUCKET = 'chapinbet-smartplay-build'
        S3_RAW_CODE = 'prueba_git'
    }
    
    stages{
    
    	stage('Copy to S3'){
            steps {
                script {
                    //Configuracion de credenciales AWS
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: '579464346048', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                        //Copia el código al buck S3 y borrar contenido local
                        sh "aws s3 cp /var/lib/jenkins/workspace/github_prueba_main s3://$S3_BUCKET/$S3_RAW_CODE/ --recursive"
                        sh "rm -rf /var/lib/jenkins/workspace/github_prueba_main/*"
                    }
                }
            }
        }

        stage('Start CodeBuild compilation'){
            steps {
                script {
                    //Configuracion de credenciales AWS
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: '579464346048', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                        //Inicia el proyecto de codebuild
                        sh "aws codebuild start-build --project-name $CODEBUILD_PROJECT"
                        sh "aws s3 rm s3://$S3_BUCKET/$S3_RAW_CODE/ --recursive"
                    }
                }
            }
        }

    }
    
}