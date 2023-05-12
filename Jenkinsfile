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
        stage('Quality Gate Status'){
                
                steps{
                    
                    script{
                        
                        waitForQualityGate abortPipeline: false, credentialsId: 'sonar-jenkins'
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
                            nexusUrl: '52.87.206.162:8081', 
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
     }   
}
