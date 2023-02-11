pipeline{
    
    agent any 
    
    stages {
        
        stage('Git Checkout'){
            
            steps{
                
                script{
                    
                    git branch: 'main', url: 'https://github.com/vikash-kumar01/mrdevops_javaapplication.git'
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
                    
                    withSonarQubeEnv(credentialsId: 'sonar-api') {
                        
                        sh 'mvn clean package sonar:sonar'
                    }
                   }
                    
                }
            }
            stage('Quality Gate Status'){
                
                steps{
                    
                    script{
                        
                        waitForQualityGate abortPipeline: false, credentialsId: 'sonar-api'
                    }
                }
            }
            stage('Upload war file to nexus'){
                
                steps{
                    
                    script{
                        def readPomVersion = readMavenPom file: 'pom.xml'
                        def nexusrepo = readPomVersion.version.endsWith("SNAPSHOT") ? "demoapp-snapshot" : "demoapp-release"
                        nexusArtifactUploader artifacts:
                        [
                            [
                                artifactId: 'springboot',
                                classifier: '',
                                file: 'target/Uber.jar',
                                type: 'jar'
                            ]
                        ],
                        credentialsId: 'nexus-auth',
                        groupId: 'com.example',
                        nexusUrl: '3.22.111.25:8081',
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        repository: 'demo-release',
                        version: "${readPomVersion.version}"
                    }
                } 
            }  
            stage("Docker build Image"){
                steps{
                    script{
                        sh 'docker image build -t $JOB_NAME:v1.$BUILD_ID'
                        Sh 'docker image tag ranchodocker/$JOB_NAME:v1.$BUILD_ID'
                        Sh 'docker image tag ranchodocker/$JOB_NAME:latest'
                    }
                }
            }
            stage('Docker build and push to Dockerhub') {
            steps{
                script{
                     withCredentials([string(credentialsID: 'dockerhub_pwd', variable: 'docker_hub_creds')]) {
                       sh 'docker login -u ranchodocker -p ${docker_creds}'
                       sh 'docker image push ranchodocker/$JOB_NAME:v1.$BUILD_ID'
                       sh 'docker image push ranchodocker/$JOB_NAME:latest' 
                     }
                   }
                }
           }
            stage('')

        }
        
}