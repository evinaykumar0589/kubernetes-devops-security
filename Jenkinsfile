pipeline {
  agent any

stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
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
      
      stage('Docker Build and Push') {
       steps {
           sh 'printenv'
           sh 'sudo docker build -t vinay0589/numeric-app:""$GIT_COMMIT"" .'
           sh 'docker push vinay0589/numeric-app:""$GIT_COMMIT""'
          }
       }
      }
   }
}
