// Define the URL of the Artifactory registry
def registry = 'https://trialp4ub3w.jfrog.io/'

pipeline {
    agent any

    environment {
        PATH = "/opt/maven/bin:$PATH"
    }

    stages {
        stage('build') {
            steps {
                echo "---------- build started ----------"
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo "---------- build completed ----------"
            }
        }

        stage('test') {
            steps {
                echo "---------- unit test started ----------"
                sh 'mvn surefire-report:report'
                echo "---------- unit test completed ----------"
            }
        }

        stage('SonarQube analysis') {
            environment {
                scannerHome = tool 'tushar-sonar-scanner'
            }
            steps {
                withSonarQubeEnv('tushar-sonarqube-server') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage('Jar Publish') {
            steps {
                script {
                    echo '<--------------- Jar Publish Started --------------->'

                    def server = Artifactory.newServer url: registry + '/artifactory', credentialsId: "artifact-id"
                    def properties = "buildId=${env.BUILD_ID},commitId=${GIT_COMMIT}"
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "jarstaging/(*)",
                                "target": "tushar-libs-release-local/{1}",
                                "flat": "false",
                                "props": "${properties}"
                                "exclusions": [ "*.sha1", "*md5"]
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

