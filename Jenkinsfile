import groovy.json.JsonOutput

def COLOR_MAP =[
    'SUCCESS': 'good',
    'FAILURE': 'danger'
]

def getBuildUser() {
  def userCause = currentBuild.rawBuild.getCause(Cause.UserIdCause)
  def upstreamCause = currentBuild.rawBuild.getCause(Cause.UpstreamCause)

  if (userCause) {
    return userCause.getUserId()
  } else if (upstreamCause) {
    def upstreamJob = Jenkins.getInstance().getItemByFullName(upstreamCause.getUpstreamProject(), hudson.model.Job.class)
    if (upstreamJob) {
      def upstreamBuild = upstreamJob.getBuildByNumber(upstreamCause.getUpstreamBuild())
      if (upstreamBuild) {
        def realUpstreamCause = upstreamBuild.getCause(Cause.UserIdCause)
        if (realUpstreamCause) {
          return realUpstreamCause.getUserId()
        }
      }
    }
  }
}

pipeline {
    agent any

    environment{
        BUILD_USER = ''
    }
    
    stages {
        stage('Compilación') {
            steps {
                sh './mvnw clean compile -e'
            }
        }
        stage('Test') {
            steps {
                sh './mvnw clean test -e'
            }
        }


       stage('Análisis Sonarqube') {
            environment {
                scannerHome = tool 'SonarScanner'
            }
            steps {
                 withSonarQubeEnv('SonarServer-1') {
                    sh './mvnw clean verify sonar:sonar -Dsonar.projectKey=lab1-mod3 -Dsonar.host.url=http://178.128.155.87:9000 -Dsonar.login=sqp_3b879c0e3e708f0dbcbfdfdf81b432e84560c4e1'
                }
            }
            
        }
        
        stage("Comprobación Quality gate") {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }  

        /*stage('Jar Code') {
            steps {
                sh './mvnw clean package -e'
            }
        }
        stage('Run Jar') {
            steps {
                sh 'nohup bash mvnw spring-boot:run &'
            }
        }
        stage('Build Deploy Code') {
            when {
                branch 'main'
            }
            steps {
                sh """
                echo "Building Artifact"
                """

                sh """
                echo "Deploying Code"
                """
            }
        }*/
        stage ('Publish Nexus'){
			when {
				branch "main"
			}
            steps{
                script {
                    pom = readMavenPom file: "pom.xml";
                    //filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    //echo "*** aqui : ${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    //def file = ${pom.artifactId}-${pom.version}.${pom.packaging}
                    echo "*** aqui : ;"//${pom.artifactId} ${pom.version} ${pom.packaging}"
                    artifactPath = "./build/${pom.artifactId}-${pom.version}.${pom.packaging}";//filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "Artifact exists: ${artifactPath}";
                         nexusPublisher nexusInstanceId: 'Nexus-1', nexusRepositoryId: 'devops-usach-nexus', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: "${artifactPath}"]], mavenCoordinate: [artifactId: 'DevOpsUsach2020', groupId: 'com.devopsusach2020', packaging: 'jar', version: pom.version]]]
                    } else {
                        echo "Artifact does not exist: ${artifactPath}";
                    }
                }
            }
        }
    }

    post{
        success{
            setBuildStatus("Build succeeded", "SUCCESS");
        }

        failure {
            setBuildStatus("Build failed", "FAILURE");
        } 

        always{
            script{
                BUILD_USER = getBuildUser()
            }

            /*slackSend channel:'#devops-equipo5',
                    color:COLOR_MAP[currentBuild.currentResult],
                    message: "*${currentBuild.currentResult}:* ${env.JOB_NAME} build ${env.BUILD_NUMBER} by ${BUILD_USER}"
                    */
         }
    }
}



void setBuildStatus(String message, String state) {
    step([
        $class: "GitHubCommitStatusSetter",
        reposSource: [$class: "ManuallyEnteredRepositorySource", url: "https://github.com/Eduardo-L-R/ms-iclab"],
        contextSource: [$class: "ManuallyEnteredCommitContextSource", context: "ci/jenkins/build-status"],
        errorHandlers: [[$class: "ChangingBuildStatusErrorHandler", result: "UNSTABLE"]],
        statusResultSource: [$class: "ConditionalStatusResultSource", results: [[$class: "AnyBuildResult", message: message, state: state]]]
    ]);
}