pipeline{
    agent any
    tools{
        jdk 'jdk 17'
        nodejs 'node 16'
        sonarQube 'SonarQube Scanner 6.1.0.4477'
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
                git branch: 'main', url: 'https://github.com/RadwaKhairy/netflix1.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                sh 'mvn clean package'
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-cred' 
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
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
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
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker'){   
                       sh "docker build --build-arg TMDB_V3_API_KEY=dec2be0775b7b7d7ed75dc7f41ff0ef0 -t netflix ."
                       sh "docker tag netflix radwakhairy/netflix1:latest "
                       sh "docker push radwakhairy/netflix1:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image radwakhairy/netflix1:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name netflix -p 8081:80 radwakhairy/netflix1:latest'
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
            to: 'radwamagdy189@gmail.com',
            attachmentsPattern: 'trivyfs.txt'
           }
        }
    }
