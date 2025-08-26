pipeline {
    agent { label "Jenkins-Agent" }
    environment {
              APP_NAME = "register-app-pipeline"
    }

    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
               steps {
                   git branch: 'main', credentialsId: 'github', url: 'https://github.com/Varshitha-DevTools/gitops-register-app.git'
               }
        }

        stage("Update the Deployment Tags") {
            steps {
                sh """
                   cat deployment.yaml
                   sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment.yaml
                   cat deployment.yaml
                """
            }
        }

        stage("Push the changed deployment file to Git") {
            steps {
                sh """
                   git config --global user.name "Varshitha-DevTools"
                   git config --global user.email "varshithag@devtools.in"
                   git add deployment.yaml
                   git commit -m "Updated Deployment Manifest"
                """
                withCredentials([gitUsernamePassword(credentialsId: 'Org', gitToolName: 'Default')]) {
                  sh "git push https://github.com/Varshitha-DevTools/gitops-register-app.git main"
                }
            }
        }
      
    }

     post {
    failure {
    emailext (
        body: """
            <h3>Build Failed</h3>
            <p>Job: ${env.JOB_NAME}</p>
            <p>Build Number: ${env.BUILD_NUMBER}</p>
            <p>Status: FAILURE</p>
        """,
        subject: "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - Failed",
        mimeType: 'text/html',
        to: "varshithag@devtools.in"
    )


        // Create GitHub Issue
        script {
            withCredentials([string(credentialsId: 'GitHub', variable: 'GITHUB_TOKEN')]) {
                def repoOwner = 'Varshitha-DevTools'   
                def repoName = 'gitops-register-app'                 
                def issueTitle = "Jenkins Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
                def issueBody = """\
                    Build URL: ${env.BUILD_URL}
                    Job: ${env.JOB_NAME}
                    Build Number: ${env.BUILD_NUMBER}
                    Result: FAILURE
                    Please check the Jenkins console output for details.
                    """.stripIndent()

                def jsonPayload = groovy.json.JsonOutput.toJson([
                    title: issueTitle,
                    body: issueBody
                ])

                // Use single quotes and shell variable to avoid secret leakage in logs
                sh """
                curl -s -H "Authorization: token \$GITHUB_TOKEN" \\
                     -H "Accept: application/vnd.github.v3+json" \\
                     -X POST \\
                     -d '${jsonPayload}' \\
                     https://api.github.com/repos/${repoOwner}/${repoName}/issues
                """
            }
        }    
    }
    
    success {
    emailext (
        body: "Job: ${env.JOB_NAME}\nBuild Number: ${env.BUILD_NUMBER}\nStatus: SUCCESS",
        subject: "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - Successful",
        to: "varshithag@devtools.in"
    )
}
}
}

