
pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'nodejs16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/prathamnandgirwar/Youtube-clone.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Youtube \
                    -Dsonar.projectKey=Youtube '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
         stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'p', usernameVariable: 'u')]) {
    // some block
                    sh "docker login -u ${env.u} -p ${env.p}"
                       sh "docker build --build-arg REACT_APP_RAPID_API_KEY=f5c3fe28c9msh069aa516d766c04p1969d1jsn085f99fefcd1 -t youtube ."
                       sh "docker tag youtube prathamnandgiwar/youtube:latest "
                       sh "docker push prathamnandgiwar/youtube:latest "
                    
                }
                 
                }
            
        }
        stage("TRIVY"){
            steps{
                sh "trivy image prathamnandgiwar/youtube:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name youtube -p 3000:3000 prathamnandgiwar/youtube:latest'
            }
        }
        
    }
}
