pipeline {
	agent any
    
    environment {
        AWS_CREDENTIALS = credentials('579464346048')
    }
    
    stages{
    
    	stage('Copy to S3'){
            steps {
                script{
                    //Copia el código al buck S3
                    sh 'aws s3 cp /var/lib/jenkins/workspace/github_prueba_main s3://chapinbet-smartplay-build/prueba_git/ --recursive'
                    sh 'rm -rf /var/lib/jenkins/workspace/github_prueba_main/*'
                }
            }
        }
    
    }
    
}