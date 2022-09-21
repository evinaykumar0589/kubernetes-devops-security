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
      stage('SonarQube - SAST') {
        steps {
          withSonarQubeEnv('SonarQube') {
            sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://devsecops-emo.eastus.cloudapp.azure.com:9000 -Dsoner.login=sqp_68af1e1ae8a6e0bc39a67a869867b0fe3250a8a0"
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
