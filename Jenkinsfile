pipeline{
    agent any
    tools{
        jdk 'jdk17'
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
        stage('Checkout From Git'){
            steps{
                git branch: 'main', url: 'https://github.com/yash509/DevSecOps-Project----Python-App-Deployment.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Python-Webapp \
                    -Dsonar.projectKey=Python-Webapp '''
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
        stage("TRIVY File scan"){
            steps{
                sh "trivy fs . > trivy-fs_report.txt"
            }
        }
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format XML ', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("Docker Build & tag"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "make image"
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image yash5090/python-system-monitoring:latest > trivy.txt"
            }
        }
        stage("Docker Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "make push"
                    }
                }
            }
        }
        stage("Deploy to container"){
            steps{
                sh "docker run -d --name python1 -p 5000:5000 yash5090/python-system-monitoring:latest"
            }
        }
        stage('Deploy to k8s'){
            steps{
                dir('K8S') {
                  withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                    sh 'kubectl apply -f deployment.yaml'
                   }
                }
            }
        }
    }
}
