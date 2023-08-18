def registry = 'https://tanishval.jfrog.io/'
pipeline {
    agent {
        node {
            label 'maven-slave'
        }
    }
environment {
    PATH = "/opt/apache-maven-3.9.4/bin:$PATH"
}


    stages {
        stage('build') {
            steps {
                sh 'mvn clean deploy -Dmaven.test.skip=true'
            }
        }

        stage("test"){
            steps{
              sh 'mvn surefire-report:report'
            }
        }


    stage('SonarQube analysis') {
      environment {
       scannerHome = tool 'galaxy-sonar-scanner';
        }
        steps{
        withSonarQubeEnv('galaxy-sonarqube-server') { // If you have configured more than one global server connection, you can specify its name
          sh "${scannerHome}/bin/sonar-scanner"
          }
        }
      }

      stage("Quality Gate"){
        steps{
            script {
        timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
          def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
          if (qg.status != 'OK') {
            error "Pipeline aborted due to quality gate failure: ${qg.status}"
          }
        }
        }
        }
      }

        stage("Jar Publish") {
            steps {
                script {
                        echo '<--------------- Jar Publish Started --------------->'
                         def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"65367a7d-d062-44bd-95d1-2f24f5a1b4b4"
                         def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                         def uploadSpec = """{
                              "files": [
                                {
                                  "pattern": "jarstaging/(*)",
                                  "target": "libs-release-local/{1}",
                                  "flat": "false",
                                  "props" : "${properties}",
                                  "exclusions": [ "*.sha1", "*.md5"]
                                }
                             ]
                         }"""
                         def buildInfo = server.upload(uploadSpec)
                         buildInfo.env.collect()
                         server.publishBuildInfo(buildInfo)
                         echo '<--------------- Jar Publish Ended --------------->'

                }
            }
        }

    }

  }


