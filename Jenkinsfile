pipeline {
     environment { 
        registry = "shailu0287/app_shailendrasharma" 
        registryCredential = 'dockerhub_id'
        dockerImage = '' 
        scannerHome = tool 'sonar_scanner_dotnet'
        containerName = "Shailendra"
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
        stage('Sonar Scan Start'){
	     when {
                branch 'master'
            }
            steps{
                withSonarQubeEnv('Test_Sonar') {
                        echo "Sonar scan start"
                        echo "${scannerHome}"
                        bat "${scannerHome}\\SonarScanner.MSBuild.exe begin /k:\"sonar-shailendrasharma\" /d:sonar.login=\"squ_b002218a4d4dd88eaf93b1e01834f4b934f33079\""
                    }
            }
        }
        stage('Build Solution') {
            steps {
                echo "Build Solution"
                  bat "\"${tool 'MSBUILD_Home'}\" nagp-devops-us.sln /p:Configuration=Release /p:Platform=\"Any CPU\" /p:ProductVersion=1.0.0.${env.BUILD_NUMBER}"
            }
        }
	stage('Release Artifect') {
	     when {
                branch 'develop'
            }
            steps {
                echo "Build Solution"
                  bat "dotnet publish -c Release -p:UseAppHost=false"
            }
        }
          stage("Test case execution") {
            when {
                branch 'master'
            }
            steps {
                 bat "dotnet test test-project\\bin\\Debug\\net5.0\\test-project.dll --collect \"XPlat Code Coverage\""
            }
        }
        stage('Sonar Scan End'){
	     when {
                branch 'master'
            }
            steps{
                withSonarQubeEnv('Test_Sonar') {
                     echo "${scannerHome}"
                      echo "sonar scan end"
                     bat "${scannerHome}\\SonarScanner.MSBuild.exe end /d:sonar.login=\"squ_b002218a4d4dd88eaf93b1e01834f4b934f33079\""
                    }
            }
        }
        stage('Build and push docker image') {
            steps{
                script {
                    docker.withRegistry('https://index.docker.io/v1/', registryCredential) {
                       // def dockerImage = docker.build("shailu0287/app_shailendrasharma/i-shailendrasharma-${BRANCH_NAME}:latest")
			dockerImage = docker.build registry + ":i-shailendrasharma-${BRANCH_NAME}"
		        dockerImage.push("i-shailendrasharma-${BRANCH_NAME}")
                        dockerImage.push("latest")
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
