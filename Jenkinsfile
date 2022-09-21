pipeline {
  agent any

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
              sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://dev-secopsdemo.eastus.cloudapp.azure.com:9000 -Dsonar.login=sqp_995477a5441556e87584e9b32f7001ff2084a1d0"
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
      stage('Kubernetes Deployment - Dev') {
       steps {
           withKubeConfig([credentialsId: 'kubeconfig']) {
           sh "sed -i 's#replace#vinay0589/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
           sh "kubectl  apply -f k8s_deployment_service.yaml"
          }
       }
     }
  } 
}
