pipeline {
    
    agent any
    
    parameters {
        booleanParam(name: 'NOCACHE', defaultValue: false, description: 'Run docker build with --no-cache')
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '3'))
    }
    
    triggers {
        pollSCM('H/15 * * * *')
    }
    
    environment {
        NS = 'openanalytics'
    }
    
    stages {
    
        stage('Docker Build') {
        
            steps {
                
                script {
                     def image = docker.build(
                        "${env.NS}/r-base",
                        ("${params.NOCACHE}".toBoolean() ? '--no-cache ' : '') + "--build-arg http_proxy='http://webproxy.openanalytics.eu:8080' --build-arg https_proxy='http://webproxy.openanalytics.eu:8080' --build-arg no_proxy='localhost,127.0.0.0,127.0.0.1,openanalytics.eu' .")
                }
                       
            }
            
        }
        
        stage('Docker Push') {
        
            steps {
            
                withDockerRegistry([
                        credentialsId: "52b706f7-b775-4fcc-83c2-3f51853250e5",
                        url: ""]) {
                        
                    sh "docker push ${env.NS}/r-base"
                    sh "docker tag ${env.NS}/r-base ${env.NS}/r-base:latest"
					sh "docker push ${env.NS}/r-base:latest"

                }
                
            }
            
        }
     
    }
    
}
