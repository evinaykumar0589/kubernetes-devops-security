pipeline {
  agent any
 
  environment {
    deploymentName = "devsecops"
    containerName = "devsecops-container"
    serviceName = "devsecops-svc"
    imageName = "vinay0589/numeric-app:${GIT_COMMIT}"
    applicationURL="http://devsecops-demo.eastus.cloudapp.azure.com"
    applicationURI="/increment/99"
  }

stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }
      stage('Unit Test') {
            steps {
              sh "mvn test"
            }
        }
      stage('Unit Test - JUnit and Jacoco') {
            steps {
              sh "mvn test"
            }
      }
      stage('Mutation Tests - PIT') {
 	  steps {
   	    sh "mvn org.pitest:pitest-maven:mutationCoverage"
          }
      }
      stage('SonarQube - SAST') {
        steps {
          withSonarQubeEnv('SonarQube') {
            sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://dev-secopsdemo.eastus.cloudapp.azure.com:9000 -Dsonar.login=sqp_995477a5441556e87584e9b32f7001ff2084a1d0"
          }
          timeout(time: 2, unit: 'MINUTES') {
            script {
             waitForQualityGate abortPipeline: true
           }
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
	 		},
                        "OPA Conftest":{
	 			sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest:latest test --policy opa-docker-security.rego Dockerfile'
	 		}
                 )
            }
      }
      stage('Docker Build and Push') {
       steps {
           withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
             sh 'printenv'
             sh 'sudo docker build -t vinay0589/numeric-app:""$GIT_COMMIT"" .'
             sh 'docker push vinay0589/numeric-app:""$GIT_COMMIT""'
            }
          }
       }
      stage('Vulnerability Scan - Kubernetes') {
       steps {
             sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-k8s-security.rego k8s_deployment_service.yaml'
           }
      }
      stage('K8S Deployment - DEV') {
       steps {
         parallel(
           "Deployment": {
             withKubeConfig([credentialsId: 'kubeconfig']) {
               sh "bash k8s-deployment.sh"
             }
           },
           "Rollout Status": {
             withKubeConfig([credentialsId: 'kubeconfig']) {
               sh "bash k8s-deployment-rollout-status.sh"
             }
           }
         )
       }
     }
  }
  post { 
        always { 
           junit 'target/surefire-reports/*.xml'
           jacoco execPattern: 'target/jacoco.exec'
           pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
           dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
           }
        }
}
