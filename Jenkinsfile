pipeline{
    agent any
    tools{
        jdk 'JDK'
        nodejs 'NodeJS'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {


        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
		stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'OWASP-check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
		stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build --build-arg TMDB_V3_API_KEY=8c12d37b4964b4e8bc708c4f720111b4 -t netflix ."
                       sh "docker tag netflix awscloudexplorer/Netflix-clone:latest "
                       sh "docker push awscloudexplorer/netflix-clone "
                    }
                }
            }
        }
		stage('Deploy to container'){
            steps{
                sh 'docker run -d --name netflix -p 8081:80 awscloudexplorer/netflix-clone:latest'
            }
        }

        stage("TRIVY"){
            steps{
                sh "trivy image awscloudexplorer/netflix-clone:latest > trivyimage.txt" 
            }
        }
    }
    post {
     always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'pramodsavvy@gmail.com',
            attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
        }
    }
}
