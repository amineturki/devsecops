

pipeline {
  
  agent any
  stages {

     stage('Build Artifact - Maven') {
       steps {
         sh "mvn clean package -DskipTests=true"
         archive 'target/*.jar'
       }
     }
        stage('UniT test') {
      steps {
       sh "mvn test"
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
               sh "sed -i 's#replace#amineturki/sringboot-app:${GIT_COMMIT}#g' k8s_PROD-deployment_service.yaml"
               sh "kubectl  apply -f k8s_PROD-deployment_service.yaml"
             }
           
                
       }
     }
    
    
    }

}
