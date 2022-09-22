

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
