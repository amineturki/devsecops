

pipeline {
  
  agent any
  stages {

     stage('Build Artifact - Maven') {
       steps {
         sh "mvn clean package -DskipTests=true"
         archive 'target/*.jar'
       }
     }
stage('unit test') {
            steps {
              sh "mvn test"
              
            }
         post {
             always {
               junit 'target/surefire-reports/*.xml'
               jacoco execPattern: 'target/jacoco.exec'
           }    
        }   
      }
        stage('Mutation Tests - PIT') {
            steps {
               sh "mvn org.pitest:pitest-maven:mutationCoverage"
     }   
          post { 
         always { 
    
           pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
     }
             }
          }
    
     stage('SonarQube SAST') {
       steps {
                     withSonarQubeEnv('SonarQube') 

               {
              sh "mvn clean verify sonar:sonar   -Dsonar.projectKey=devsecops1   -Dsonar.host.url=http://devsecops-demo-amine.eastus.cloudapp.azure.com:9000   -Dsonar.login=sqp_4a7c2b4ab2506541fff8e57b92995fcf20a0f382"  
              
            }
                  timeout(time: 2, unit: 'MINUTES') {
           script {
             waitForQualityGate abortPipeline: true
          }
        }
                     }

    }
    
        stage('Docker Build and Push') {
       steps {
         withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
           sh 'printenv'
           sh 'sudo docker build -t amineturki/sringboot-app:""$GIT_COMMIT"" .'
           sh 'docker push amineturki/sringboot-app:""$GIT_COMMIT""'
         }
       }
     }
    
         stage('K8S Deployment - DEV') {
       steps {
         
         
             withKubeConfig([credentialsId: 'kubeconfig']) {
               sh "sed -i 's#replace#amineturki/sringboot-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
               sh "kubectl  apply -f k8s_deployment_service.yaml"
             }
           
                
       }
     }
    
    
    }

}
