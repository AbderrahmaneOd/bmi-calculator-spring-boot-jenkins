pipeline {
    agent any 
    
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    
    
    stages{
        
        stage('Git Checkout'){
            steps{
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/AbderrahmaneOd/bmi-calculator-spring-boot-jenkins'
            }
        }
        
        stage('Compile'){
            steps{
                sh "mvn clean compile"
            }
        }
        
         stage('Test Cases'){
            steps{
                sh "mvn test"
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                sh ''' mvn sonar:sonar \
                        -Dsonar.projectKey=BMI-Calculator-SpringBoot \
                        -Dsonar.projectName='BMI-Calculator-SpringBoot' \
                        -Dsonar.host.url=http://localhost:9000 \
                        -Dsonar.token=sqp_89927fbefa9d371ed4105c285966c228dff69760
                     '''
            }
        }
        
        stage('OWASP Dependency Check'){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format HTML ', odcInstallation: 'db-check'
            }
        }
        
         stage('Build'){
            steps{
                sh " mvn clean package"
            }
        }

        
        stage('Docker Build & Push'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'DockerHub-Token', toolName: 'docker') {
                        def imageName = "bmi-calculator-springboot"
                        def buildTag = "${imageName}:${BUILD_NUMBER}"
                        def latestTag = "${imageName}:latest"  // Define latest tag
                        
                        sh "docker build -t ${imageName} -f Dockerfile ."
                        sh "docker tag ${imageName} abdeod/${buildTag}"
                        sh "docker tag ${imageName} abdeod/${latestTag}"  // Tag with latest
                        sh "docker push abdeod/${buildTag}"
                        sh "docker push abdeod/${latestTag}"  // Push latest tag
                    }
                        
                }
            }
        }
       
        
        stage('Vulnerability scanning'){
            steps{
                sh "trivy image abdeod/bmi-calculator-springboot:latest"
            }
        }

        
        stage('Deploy'){
            steps{
                sh 'docker-compose up -d'
            }
        }
   
    }
}