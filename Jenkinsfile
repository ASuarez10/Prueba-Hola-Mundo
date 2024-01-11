pipeline {
	agent any
    
    environment {
        //Credenciales
        AWS_CREDENTIALS_ST = '579464346048' //Credenciales AWS Security Tooling
        AWS_CREDENTIALS_CB = 'chapinbet_credentials' //Credenciales AWS ChapinBet

        //Constantes info Secutiry Tooling
        CODEBUILD_PROJECT = 'jenkins-prueba' //CodeBuild project name
        S3_BUCKET = 'chapinbet-smartplay-build' //Nombre del bucket de S3
        S3_RAW_CODE = 'prueba_git' //Nombre de la carpeta donde estará el código original que se baja desde el repositorio
        S3_ARTIFACTS = 'Artefactos' //Nombre de la carpeta donde se guardan los artefactos generados por el CodeBuild
        S3_CODEBUILD_ARTIFACT = 'prueba_git_artefacto' //Nombre del artefacto generado por el CodeBuild

        //Constantes ChapinBet
        CODEDEPLOY_APPLICATION_NAME = 'chapinbet-smartplay' //Nombre de la aplicación de CodeDeploy
        CODEDEPLOY_DEPLOYMENT_GROUP = 'GDI-Frontend' //Nombre del GDI de CodeDeploy
    }
    
    stages{
    
    	stage('Copy to S3'){
            steps {
                script {
                    //Configuracion de credenciales AWS
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: "$AWS_CREDENTIALS_ST", secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                        //Copia el código al buck S3 y borrar contenido local
                        sh "aws s3api put-object --bucket $S3_BUCKET --key $S3_RAW_CODE/"
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

                        def buildId = sh(script: "aws codebuild start-build --project-name $CODEBUILD_PROJECT --query 'build.id' --output text", returnStdout: true).trim()
                        echo "Build ID: $buildId"

                        def buildComplete = false

                        //Esperar hasta que la compilacion este completa
                        while (!buildComplete) {
                        // Obtener el estado de la compilación
                            def buildStatus = sh(script: 'aws codebuild batch-get-builds --ids ' + buildId + ' --query "builds[0].buildComplete" --output text', returnStdout: true).trim()

                            echo "Estado de la compilacion: ${buildStatus}"

                            if (buildStatus == 'True') {
                                echo 'La compilación ha sido completada.'
                                buildComplete = true
                            } else {
                                echo 'Esperando a que la compilación se complete...'
                                sleep time: 1, unit: 'SECONDS'  // Ajusta el intervalo de espera según tus necesidades
                            }
                        }

                    }
                }
            }
        }

        stage('Deploy application'){
            steps {
                script {
                    //Configuracion de credenciales AWS
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: "$AWS_CREDENTIALS_CB", secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                        //Inicia el despliegue
                        //sh "aws deploy create-deployment --application-name $CODEDEPLOY_APPLICATION_NAME --deployment-group-name $CODEDEPLOY_DEPLOYMENT_GROUP --s3-location bucket=$S3_BUCKET,key=$S3_ARTIFACTS/$S3_CODEBUILD_ARTIFACT,bundleType=zip --ignore-application-stop-failures"
                    
                        def deploymentJson = sh(script: "aws deploy create-deployment --application-name $CODEDEPLOY_APPLICATION_NAME --deployment-group-name $CODEDEPLOY_DEPLOYMENT_GROUP --s3-location bucket=$S3_BUCKET,key=$S3_ARTIFACTS/$S3_CODEBUILD_ARTIFACT,bundleType=zip", returnStdout: true).trim()
                    
                        // Parsea el JSON para obtener el valor de deploymentId
                        def deploymentId = new groovy.json.JsonSlurper().parseText(deploymentJson).deploymentId

                        def deploymentComplete = false

                         // Esperar hasta que el despliegue esté completo
                        while (!deploymentComplete) {
                            // Obtener información sobre el estado del despliegue
                            echo "Deployment ID: ${deploymentId}"

                            def deploymentStatus = sh(script: 'aws deploy get-deployment --deployment-id ' + deploymentId + ' --query "deploymentInfo.status" --output text', returnStdout: true).trim()

                            echo "Estado del despliegue: ${deploymentStatus}"

                            if (deploymentStatus == 'Succeeded' || deploymentStatus == 'Failed') {
                                echo 'El despliegue ha sido completado.'
                                deploymentComplete = true
                            } else {
                                echo 'Esperando a que el despliegue se complete...'
                                sleep time: 1, unit: 'SECONDS'  // Ajusta el intervalo de espera según tus necesidades
                            }
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
                    sh "aws s3 rm s3://$S3_BUCKET/$S3_RAW_CODE/ --recursive"
                }
            }
        }
    }
    
}