pipeline {
     environment { 
        registry = "shailu0287/app_shailendrasharma" 
        registryCredential = 'dockerhub_id'
        dockerImage = '' 
        scannerHome = tool 'sonar_scanner_dotnet'
        containerName = "Shailendra"
        PROJECT_ID = 'nagpjenkinsdevops'
        CLUSTER_NAME = 'nagpcluster-1'
        LOCATION = 'us-central1-c'
        CREDENTIALS_ID = 'NAGPJenkinsDevops'
        VSTest_Home = tool 'VSTest_Home'
    }
    agent any

    stages {
        stage('Code Checkout') {
            steps {
                 cleanWs()
                   echo "Github checkout"
                  checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/shailu0287/app_shailendrasharma.git']]])
            }
        }
        stage('Nuget Restore') {
            steps {
                  echo "nuget restore"
                  bat 'dotnet restore \"nagp-devops-us.sln\"'
            }
        }
     
        stage('Build Solution') {
            steps {
                echo "Build Solution"
                  bat "\"${tool 'MSBUILD_Home'}\" nagp-devops-us.sln /p:Configuration=Release /p:Platform=\"Any CPU\" /p:ProductVersion=1.0.0.${env.BUILD_NUMBER}"
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
         stage('Deploy to GKE') {
            steps{
                echo "Deployment started ..."
                bat "kubectl apply -f .\\K8s\\"
            }
        }
        
    }
}
