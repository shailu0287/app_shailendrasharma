pipeline {
     environment { 
        registry = "shailu0287/jenkins" 
        registryCredential = 'dockerhub_id'
        dockerImage = '' 
        scannerHome = tool 'sonarscanner'
        containerName = "Shailendra"
        PROJECT_ID = 'nagpjenkinsdevops'
        CLUSTER_NAME = 'nagpcluster-1'
        LOCATION = 'us-central1-c'
        CREDENTIALS_ID = 'NAGPJenkinsDevops'
    }
    agent any

    stages {
        stage('Code Checkout') {
            steps {
                 cleanWs()
                   echo "Github checkout"
                  checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://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@github.com/shailu0287/JenkinsTest.git']]])
            }
        }
        stage('Nuget Restore') {
            steps {
                  echo "nuget restore"
                  bat 'dotnet restore \"WebApplication4.sln\"'
            }
        }
        stage('Sonar Scan Start'){
            steps{
                withSonarQubeEnv('sonarserver') {
                        echo "Sonar scan start"
                        echo "${scannerHome}"
                        bat "${scannerHome}\\SonarScanner.MSBuild.exe begin /k:\"Pan33r\" /d:sonar.login=\"squ_d23f21a836ee8b5fb7815ebc3cbba256c6d4537c\""
                    }
            }
        }
        stage('Build Solution') {
            steps {
                echo "Build Solution"
                  bat "\"${tool 'msbuilddefault'}\" WebApplication4.sln /p:Configuration=Release /p:Platform=\"Any CPU\" /p:ProductVersion=1.0.0.${env.BUILD_NUMBER}"
            }
        }
        stage('Sonar Scan End'){
            steps{
                withSonarQubeEnv('sonarserver') {
                     echo "${scannerHome}"
                      echo "sonar scan end"
                     bat "${scannerHome}\\SonarScanner.MSBuild.exe end /d:sonar.login=\"squ_d23f21a836ee8b5fb7815ebc3cbba256c6d4537c\""
                    }
            }
        }
        stage('Building docker image') {
            steps{
                script {
                    echo "Building docker image"
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }
        stage('Containers'){
            parallel{
                stage("Run PreContainer Checks"){
                    environment{
                        containerID = "${bat(script: 'docker ps -a -q -f name="c-Shailendra-master"', returnStdout: true).trim().readLines().drop(1).join("")}"
                    }
                    steps{
                        script{
                            echo "Run PreContainer Checks"
                            echo env.containerName
                            echo "containerID is "
                            echo env.containerID
                            
                            if(env.containerID != null){
                                echo "Stop container and remove from stopped container list too"
                                bat "docker stop ${env.containerID} && docker rm ${env.containerID}"
                            }                         
                        }
                    }
                }
                stage("Publish Docker Image to DockerHub"){
                    steps{
                        script {
                            echo "Pushing docker image to docker hub"
                            docker.withRegistry( '', registryCredential ) {
                                dockerImage.push("$BUILD_NUMBER")
                                dockerImage.push('latest')   
                            }   
                        }                 
                    }
                }
            }    
        } 
         stage('Docker Deployment'){
            steps{
                echo "${registry}:${BUILD_NUMBER}"
                echo "Docker Deployment by using docker hub's image"
                bat "docker run -d -p 7200:80 --name c-${containerName}-master ${registry}:${BUILD_NUMBER}"
            }

        }
        stage('Deploy to GKE') {
            steps{
                echo "Deployment started ..."
                step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'deployment.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
            }
        }
    }
}
