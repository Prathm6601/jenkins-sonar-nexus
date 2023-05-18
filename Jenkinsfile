pipeline{
    
    agent any 
    
    stages {
        
        stage('Git Checkout'){
            
            steps{
                
                script{
                    
                    git branch: 'main', url: 'https://github.com/Prathm6601/jenkins-sonar-nexus.git'
                }
            }
        }
        stage('UNIT testing'){
            
            steps{
                
                script{
                    
                    sh 'mvn test'
                }
            }
        }
        stage('Integration testing'){
            
            steps{
                
                script{
                    
                    sh 'mvn verify -DskipUnitTests'
                }
            }
        }
        stage('Maven build'){
            
            steps{
                
                script{
                    
                    sh 'mvn clean install'
                }
            }
        }
        stage('Static code analysis'){
            
            steps{
                
                script{
                    
                    withSonarQubeEnv(credentialsId: 'sonar-jenkins') {
                        
                        sh 'mvn clean package sonar:sonar'
                    }
                   }
                    
                }
            }
       
        stage('upload war file to nexus'){
       
                steps{
              
                    script{
                        def readPomVersion = readMavenPom file: 'pom.xml'
                        def nexusRepo = readPomVersion.version.endsWith("SNAPSHOT") ? "ait-snapshot" : "ait-release"
                        nexusArtifactUploader artifacts: 
                            [
                                [
                                    artifactId: 'spring-boot-starter-parent', 
                                    classifier: '', 
                                    file: 'target/Uber.jar', type: 'jar'
                                ]
                            ], 
                            credentialsId: 'nexus-a', 
                            groupId: 'com.example', 
                            nexusUrl: '35.175.142.195:8081', 
                            nexusVersion: 'nexus3', 
                            protocol: 'http', 
                            repository: nexusRepo, 
                            version: "${readPomVersion.version}"
                    }
                }
        }
        stage('Docker Image Build'){
            steps{
                script {
                    sh 'docker image build -t $JOB_NAME:v1.$BUILD_ID .'
                    sh 'docker image tag $JOB_NAME:v1.$BUILD_ID prathm6601/$JOB_NAME:v1.$BUILD_ID'
                    sh 'docker image tag $JOB_NAME:v1.$BUILD_ID prathm6601/$JOB_NAME:latest'
                }
            }
        }
        stage('push image to the dockerHub'){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_cred', variable: 'docker_hub')]){
                        sh 'docker login -u prathm6601 -p ${docker_hub}'
                        sh 'docker image push prathm6601/$JOB_NAME:v1.$BUILD_ID'
                        sh 'docker image push prathm6601/$JOB_NAME:latest'
                    }
                }
            }
        }
        stage('docker build andpush image to the nexus repo'){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus-cred', variable: 'nexus-cred')]){
                        sh 'docker image build -t 35.175.142.195:8083/$JOB_NAME:v1.$BUILD_ID .'
                        sh 'docker login -u admin -p prathm 35.175.142.195:8083'
                    }
                }
            }
        }
     }   
}
