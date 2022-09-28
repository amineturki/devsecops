

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
               sh "mvn clean verify sonar:sonar \
  -Dsonar.projectKey=devsecops \
  -Dsonar.host.url=http://devsecops-demo-amine.eastus.cloudapp.azure.com:9000 \
  -Dsonar.login=sqp_4ded4184a0f8bdc4260a25ecac8a94e9b633c30e"  
              
            }
          
                     }

    }
    
 stage('Vulnerability Scan - Docker') {
       steps {
         parallel(
         	"Dependency Scan": {
         		sh "mvn dependency-check:check"
	 		},
	 		"Trivy Scan":{
	 			sh "bash trivy-docker-image-scan.sh"
	 		} ,
	 		"OPA Conftest":{
	 			sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-docker-security.rego Dockerfile'
	 		}   	
       	)
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
	
	 post {
             always {
               junit 'target/surefire-reports/*.xml'
               jacoco execPattern: 'target/jacoco.exec'
	       dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
	       //pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
           }    
        }   

}


