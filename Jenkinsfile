def COLOR_MAP = [
    'FAILURE' : 'danger',
    'SUCCESS' : 'good'
]

pipeline{
    agent any
    tools{
        nodejs 'nodejs21'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('Clean Workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/ManjuNK/Youtube-clone-app.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=youtube \
                    -Dsonar.projectKey=youtube '''
                }
            }
        }
        stage("Quality Gate"){
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
        //stage('OWASP FS SCAN') {
        //    steps {
        //      dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
        //        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        //    }
       // }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build --build-arg REACT_APP_RAPID_API_KEY=aaad0c5d01mshb5bf4d31fab71b2p10df6djsn3eb8d43d4cda -t youtube ."
                       sh "docker tag youtube manjunk/youtube:latest "
                       sh "docker push manjunk/youtube:latest "
                    }
                }
            }
        }
        stage("TRIVY Image Scan"){
            steps{
                sh "trivy image manjunk/youtube:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to Container'){
            steps{
                sh 'docker run -d --name youtube422 -p 3000:3000 manjunk/youtube:latest'
            }
        }
        stage('Deploy to Kubernets'){
            steps{
                script{
                    withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                        sh 'kubectl apply -f deployment.yml'
                    }
                }
            }
        }

    }
    post {
         always {
             echo 'Slack Notifications'
             slackSend (
                 channel: '#jenkins',
                 color: COLOR_MAP[currentBuild.currentResult],
                 message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} \n build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
               )
           }
       }
      
   }
