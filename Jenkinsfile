pipeline {
    
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '3'))
    }
    
    triggers {
        pollSCM('H/15 * * * *')
    }
    
    environment {
        REG_OA = 'registry.openanalytics.eu'
        REPO_OA = 'public'
        REPO_HUB = 'openanalytics'
        IMAGE = "r-base"
    }
    
    stages {
    
        stage('Docker Build') {
        
            steps {
            
                sh """
                docker build -t ${env.IMAGE} \
                    --build-arg http_proxy='http://webproxy.openanalytics.eu:8080' \
                    --build-arg https_proxy='http://webproxy.openanalytics.eu:8080' \
                    --build-arg no_proxy='localhost,127.0.0.0,127.0.0.1,openanalytics.eu' \
                    .
                """
            
            }
            
        }
        
        stage('Docker Push (versioned)') {
        
            steps {
            
                withDockerRegistry([
                        credentialsId: "jenkins-portus-token",
                        url: "https://registry.openanalytics.eu"]) {
                        
                    sh """
                    export VERSION=\$(docker inspect ${env.IMAGE} -f '{{ index .Config.Labels.version }}')
                    docker tag ${env.IMAGE} ${env.REG_OA}/${env.REPO_OA}/${env.IMAGE}:\$VERSION
                    docker push ${env.REG_OA}/${env.REPO_OA}/${env.IMAGE}:\$VERSION
                    """

                }
                
                withDockerRegistry([
                         credentialsId: "openanalytics-dockerhub",
                         url: "https://index.docker.io/v1/"]) {
                     
                     sh """
                     export VERSION=\$(docker inspect ${env.IMAGE} -f '{{ index .Config.Labels.version }}')
                     docker tag ${env.IMAGE} ${env.REPO_HUB}/${env.IMAGE}:\$VERSION
                     docker push ${env.REPO_HUB}/${env.IMAGE}:\$VERSION
                     """
                     
                }
            }
        }
        
        stage ('Docker Push (latest)') {
            when {
                branch "master"
            }
            steps {
                withDockerRegistry([
                        credentialsId: "jenkins-portus-token",
                        url: "https://registry.openanalytics.eu"]) {
                    sh """
                    docker tag ${env.IMAGE} ${env.REG_OA}/${env.REPO_OA}/${env.IMAGE}:latest
                    docker push ${env.REG_OA}/${env.REPO_OA}/${env.IMAGE}:latest
                    """
                }
                withDockerRegistry([
                         credentialsId: "openanalytics-dockerhub",
                         url: "https://index.docker.io/v1/"]) {
                    sh """
                    docker tag ${env.IMAGE} ${env.REPO_HUB}/${env.IMAGE}:latest
                    docker push ${env.REPO_HUB}/${env.IMAGE}:latest
                    """
                }
            }
        }
     
    }
    
}
