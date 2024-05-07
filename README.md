this is the Jenkins file-scripted pipeline 
pipeline {
    agent any

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "M2_HOME"
    }

    stages {
        stage('Build') {
            steps {
                // Get some code from a GitHub repository
                git url: 'https://github.com/NishaJomy/CICD.git', branch: 'master'

                // Navigate to the directory containing the pom.xml file
                dir('cicddemo') {
                    // Run Maven commands in the project directory
                    bat "mvn -Dmaven.test.failure.ignore=true clean package"
                }
            }

            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                success {
                    junit '**/target/surefire-reports/TEST-*.xml'
                    archiveArtifacts 'cicddemo/target/*.jar'
                }
            }
        }
        stage('Build docker image'){
            steps{
                script{
                    dir('cicddemo') {
                        bat 'docker build -t nishageorge/devopsintegration .'
                    }
                }
            }
        }
        
     stage('Push image to Hub'){
            steps{
                script{
                  withCredentials([string(credentialsId: 'dockerhpwd', variable: 'dockerhpwd')])  
   
                    {
                        bat "docker login -u nishageorge -p ${dockerhpwd}"
                        bat "docker push nishageorge/devops-integration"

                    }
                   
                }
            }
        }
         stage('Deploy to k8s'){
            steps{
                script{
                    kubernetesDeploy (configs: 'deploymentservice.yaml',kubeconfigId: 'k8sconfigpwd')
                }
            }
        }

}
}
