pipeline {
     environment { 
        registry = "shailu0287/app_shailendrasharma" 
        registryCredential = 'dockerhub_id'
        dockerImage = '' 
        scannerHome = tool 'sonar_scanner_dotnet'
	SONAR_PROJECT_KEY= 'sonar-shailendrasharma'
	MSBUILD = tool 'MSBUILD_Home'
    }
    agent any

    stages {
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
			bat "${scannerHome}\\SonarScanner.MSBuild.exe begin /k:\"${SONAR_PROJECT_KEY}\" /d:sonar.login=\"squ_b002218a4d4dd88eaf93b1e01834f4b934f33079\""
                    }
            }
        }
        stage('Build Solution') {
            steps {
                echo "Build Solution"
                  bat "\"${MSBUILD}\" nagp-devops-us.sln /p:Configuration=Release /p:Platform=\"Any CPU\" /p:ProductVersion=1.0.0.${env.BUILD_NUMBER}"
            }
        }
	stage('Release Artifect') {
	     when {
                branch 'develop'
            }
            steps {
                echo "Release Artifect"
                  bat "dotnet publish -c Release -p:UseAppHost=false"
            }
        }
          stage("Test case execution") {
            when {
                branch 'master'
            }
            steps {
		 echo "Unit test run"
                 bat "dotnet test test-project\\bin\\Debug\\net5.0\\test-project.dll --collect \"XPlat Code Coverage\""
            }
        }
        stage('Sonar Scan End'){
	     when {
                branch 'master'
            }
            steps{
                withSonarQubeEnv('Test_Sonar') {
                      echo "sonar scan end"
                     bat "${scannerHome}\\SonarScanner.MSBuild.exe end /d:sonar.login=\"squ_b002218a4d4dd88eaf93b1e01834f4b934f33079\""
                    }
            }
        }
        stage('Build and push docker image') {
            steps{
                script {
                    docker.withRegistry('https://index.docker.io/v1/', registryCredential) {
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
