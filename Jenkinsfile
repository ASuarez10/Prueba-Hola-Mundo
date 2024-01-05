pipeline {
	agent any
    
    environment {
        //Credenciales
        AWS_CREDENTIALS_ST = '579464346048' //Credenciales AWS Security Tooling
        AWS_CREDENTIALS_CB = 'chapinbet_credentials' //Credenciales AWS ChapinBet

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
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: "$AWS_CREDENTIALS_ST", secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
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
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: "$AWS_CREDENTIALS_ST", secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                        //Inicia el proyecto de codebuild
                        //sh "aws codebuild start-build --project-name $CODEBUILD_PROJECT"
                        //sh "aws s3 rm s3://$S3_BUCKET/$S3_RAW_CODE/ --recursive"

                        def buildId = sh(script: "aws codebuild start-build --project-name $CODEBUILD_PROJECT --query 'build.id' --output text", returnStdout: true).trim()
                        echo "Build ID: $buildId"
                        
                        // Espera hasta que la compilación de CodeBuild haya finalizado
                        sh "aws codebuild wait build-completed --id $buildId"

                        // Captura el estado de salida de la compilación de CodeBuild
                        def buildStatus = sh(script: "aws codebuild batch-get-builds --ids $buildId --query 'builds[0].buildStatus' --output text", returnStatus: true).trim()

                        // Si la compilación de CodeBuild falla, arroja una excepción
                        if (buildStatus != 'SUCCEEDED') {
                            error "La compilación de CodeBuild ha fallado."
                        }
                    }
                }
            }
        }

    }

    post {
        always {
            script {
                // Configuracion de credenciales AWS para la limpieza después de CodeBuild
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: "$AWS_CREDENTIALS_ST", secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                    // Borra el contenido de la carpeta en S3
                    sh "aws s3 rm s3://$S3_BUCKET/$S3_RAW_CODE/*"
                }
            }
        }
    }
    
}