pipeline {
  agent any

  tools {
    jdk 'Java'
    maven 'maven' //đổi tên
  }

  stages {
    stage('Build && Test') {
      steps {
        withMaven(maven : 'maven') {
          sh "mvn package"
        }
      }
    }
    stage ('OWASP Dependency-Check Vulnerabilities') {
      steps {
        dependencyCheck additionalArguments: ''' 
          -o "./" 
          -s "./"
          -f "ALL" 
          --prettyPrint''', odcInstallation: 'DependencyCheck'

        dependencyCheckPublisher pattern: 'dependency-check-report.xml'
      }
    }
    stage('SonarQube analysis') {
      environment {
                scannerHome = tool 'SonarQubeScanner'
      }
      steps {
        withSonarQubeEnv('sonarqube') {
          withMaven(maven : 'maven') {
            sh 'mvn sonar:sonar -Dsonar.dependencyCheck.jsonReportPath=target/dependency-check-report.json -Dsonar.dependencyCheck.xmlReportPath=target/dependency-check-report.xml -Dsonar.dependencyCheck.htmlReportPath=target/dependency-check-report.html'
          }
        }
      }
    }
    stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

    stage('Push container to docker hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
          withMaven(maven : 'maven') {
            sh "mvn jib:build"
          }
        }
      } 
    }

    stage('Run container') {
          steps {
                sh "docker run --network Final_Milestone -p 8080:8080 --ip 172.18.0.100 -d --name milestone buiducanh68/spring-boot-demo"
          } 
        }
    stage('OWASP ZAP'){
                sh "docker run -t owasp/zap2docker-stable http://localhost:8082 --network Final_Milestone -d buiducanh68/spring-boot-demo"
    }
  }
}
