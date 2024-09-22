pipeline {
  agent any

  environment {
    DOCKERHUB_CREDENTIALS = credentials('DOCKER_HUB_CREDENTIAL')
    VERSION = "${env.BUILD_ID}"

  }

  tools {
    maven "maven"
  }

  stages {

    stage('Maven Build'){
        steps{
        sh 'mvn clean package  -DskipTests'
        }
    }

     stage('Run Tests') {
      steps {
        sh 'mvn test'
      }
    }

    stage('SonarQube Analysis') {
  steps {
    sh 'mvn clean org.jacoco:jacoco-maven-plugin:prepare-agent install sonar:sonar -Dsonar.host.url=http://52.56.190.119:9000/ -Dsonar.login=squ_4870ef594776e5ec5792df5e708e8445c30eb60e'
  }
}


   stage('Check code coverage') {
            steps {
                script {
                    def token = "squ_4870ef594776e5ec5792df5e708e8445c30eb60e"
                    def sonarQubeUrl = "http://52.56.190.119:9000/api"
                    def componentKey = "com.codeddecode:restaurantlisting"
                    def coverageThreshold = 80.0

                    def response = sh (
                        script: "curl -H 'Authorization: Bearer ${token}' '${sonarQubeUrl}/measures/component?component=${componentKey}&metricKeys=coverage'",
                        returnStdout: true
                    ).trim()

                    def coverage = sh (
                        script: "echo '${response}' | jq -r '.component.measures[0].value'",
                        returnStdout: true
                    ).trim().toDouble()

                    echo "Coverage: ${coverage}"

                    if (coverage < coverageThreshold) {
                        error "Coverage is below the threshold of ${coverageThreshold}%. Aborting the pipeline."
                    }
                }
            }
        } 


      stage('Docker Build and Push') {
//       steps {
//           sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
//           sh 'docker build -t somnath2175/restaurant-listing-service:${VERSION} .'
//           sh 'docker push somnath2175/restaurant-listing-service:${VERSION}'
//       }

      steps {
        // Use sudo to run Docker as root
        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | sudo docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
        sh 'sudo docker build -t somnath2175/restaurant-listing-service:${VERSION} .'
        sh 'sudo docker push somnath2175/restaurant-listing-service:${VERSION}'
      }

    } 


     stage('Cleanup Workspace') {
      steps {
        deleteDir()
       
      }
    }



    stage('Update Image Tag in GitOps') {
      steps {
         checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[ credentialsId: 'git-ssh', url: 'https://github.com/FullStackProjectAngularWithJava/deployment-folder.git']])
        script {
       sh '''
          sed -i "s/image:.*/image: somnath2175\\/restaurant-listing-service:${VERSION}/" aws/restaurant-manifest.yml
        '''
          sh 'git checkout main'
          sh 'git add .'
          sh 'git commit -m "Update image tag"'
        sshagent(['git-ssh'])
            {
                  sh('git push')
            }
        }
      }
    }

  }

}


