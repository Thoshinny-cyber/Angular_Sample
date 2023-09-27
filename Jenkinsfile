pipeline{
    agent any

    // tools {
    //   nodejs 'node'
    // }

    environment {
      DOCKER_TAG = getVersion()
      DOCKER_CRED= credentials('docker_hub1')
    }

    stages{
        stage('SCM'){
            steps{
                 deleteDir()
                git 'https://github.com/Thoshinny-cyber/Angular_Sample.git'
            }
        }
          stage('Build') {
            steps {
                sh 'tar czf Angular.tar.gz *'
            }
        }
        
   // }

   // post {
       // always {
            // Proceed with Docker Build stage or other actions after approval
            stage('Docker Build') {
                steps {
                    sh "echo ${DOCKER_TAG}"
                    // Add your Docker build and push steps here
                            sshPublisher(publishers: [
            sshPublisherDesc(
                configName: 'dockerhost',
                transfers: [
                    sshTransfer(
                        cleanRemote: false,
                        excludes: '',
                        execCommand: """cd /opt/docker1; 
                                        tar -xf Angular.tar.gz; 
                                        docker build . -t thoshinny/angularapp:${DOCKER_TAG}
                                        docker push thoshinny/angularapp:${DOCKER_TAG}
                                        """,
                        execTimeout: 2000000,
                        flatten: false,
                        makeEmptyDirs: false,
                        noDefaultExcludes: false,
                        patternSeparator: '[, ]+$',
                        remoteDirectory: '//opt//docker1',
                        remoteDirectorySDF: false,
                        removePrefix: '',
                        sourceFiles: '**/*.gz'
                    )
                ],
                usePromotionTimestamp: false,
                useWorkspaceInPromotion: false,
                verbose: true
            )
        ])
                  script {
                    if (currentBuild.resultIsBetterOrEqualTo('SUCCESS')) {
                        // Send success email to manager for approval
                        sendApprovalEmail('SUCCESS')
                    } else {
                        // Send failure email to manager for previous build
                        sendFailureEmail('FAILURE')
                    }
                }
            }
            }
        //}
    //}
// stage('Approval') {
//             steps {
//                 // Send an email notification to the manager for approval
//                script{
//                  def previousCommit = sh(returnStdout: true, script: 'git rev-parse HEAD~1')
//             //       if (previousCommit != 0) {
//             //     error("Failed to get previous commit hash.")
//             // }
//                  def currentCommit = sh(returnStdout: true, script: 'git rev-parse HEAD')
//                  def changes =  sh(script: 'git show --name-status HEAD^', returnStdout: true).trim()
//                  def authorEmail = sh(script: 'git log -1 --format="%ae"', returnStdout: true).trim()
//                  def approvalMail = """
//                     Build ${env.BUILD_NUMBER} of ${env.JOB_NAME} has completed.
//                     Commit ID: ${env.GIT_COMMIT}
//                     Previous Commit ID: ${previousCommit}
//                     Docker tag: ${env.DOCKER_TAG}
//                     Source Path: ${env.WORKSPACE}
//                     Author: ${authorEmail}
//                     Date: ${env.BUILD_TIMESTAMP}
//                     Build Result: ${currentBuild.result}
//                     Please review and approve or reject this build.
//                     BUILD URL: ${env.BUILD_URL}
//                     """
//                  def mailSubject =  "Approval Required for Docker Deployment - ${currentBuild.displayName}"
//                  def gitDiffOutput = sh(script: "git diff HEAD~1 ${currentCommit}", returnStdout: true)
// // Combine 'changes' and 'gitDiffOutput' and write to 'changelog.txt'
//                  def combinedContent = changes + "\n\n" + gitDiffOutput
//                  writeFile(file: 'changelog.txt', text: combinedContent)
//                 if (gitDiffOutput.isEmpty()) {
//                 error("No changes found between commits.")
//             }
                
//                 emailext (
//                     subject: mailSubject,
//                     body: approvalMail,
//                     mimeType: 'text/plain',
//                     to: 'thoshlearn@gmail.com',
//                     attachmentsPattern: 'changelog.txt'
//                     //attachmentsPattern: "${currentBuild.changeSets.fileChanges.file}", // Attach the changelog as a text file
//                     //attachLog: true, // Attach the build log
//                    // replyTo: currentBuild.upstreamBuilds[0]?.actions.find { it instanceof hudson.model.CauseAction }?.cause.upstreamProject
//                 )

//                 // Wait for manager approval
//                 timeout(time: 10, unit: 'MINUTES') {
//                     input message: 'Waiting for Manager Approval'
//                 }
//                     }
//             }
//         }
          

        stage('Docker Deploy') {
            steps {
                script {
                    def ansiblePlaybookContent = '''
                    - hosts: dev
                      become: True

 

                      tasks:
                        - name: Install python pip
                          yum:
                            name: python-pip
                            state: present

 

                        - name: Install docker-py python module
                          pip:
                            name: docker-py
                            state: present

 

                        - name: Start the container
                          docker_container:
                            name: nodecontainer
                            image: "thoshinny/angularapp:{{ DOCKER_TAG }}"
                            state: started
                            published_ports:
                              - 0.0.0.0:80:80
                    '''

 

                    writeFile(file: 'inline_playbook.yml', text: ansiblePlaybookContent)

 

                   def ansibleInventoryContent = '''[dev]
                    172.31.42.16 ansible_user=ec2-user
                    '''

 

                    writeFile(file: 'dev.inv', text: ansibleInventoryContent)

 

   
                    ansiblePlaybook(
                        inventory: 'dev.inv',
                        playbook: 'inline_playbook.yml',
                        extras: "-e DOCKER_TAG=${DOCKER_TAG}",
                        credentialsId: 'dev-server',
                        installation: 'ansible'
                    )

              }
            }
        }

 

}

}

 

 

def getVersion(){
    def commitHash = sh label: '', returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}

def sendApprovalEmail(buildStatus) {
    def previousCommit = sh(returnStdout: true, script: 'git rev-parse HEAD~1')
    def currentCommit = sh(returnStdout: true, script: 'git rev-parse HEAD')
    def changes = sh(script: 'git show --name-status HEAD^', returnStdout: true).trim()
    def authorEmail = sh(script: 'git log -1 --format="%ae"', returnStdout: true).trim()

    def mailSubject = "SUCCESS Notification - Approval Required for Docker Deployment - ${currentBuild.displayName}"
    def gitDiffOutput = sh(script: "git diff HEAD~1 ${currentCommit}", returnStdout: true)

    // Combine 'changes' and 'gitDiffOutput' and write to 'changelog.txt'
    def combinedContent = changes + "\n\n" + gitDiffOutput
    writeFile(file: 'changelog.txt', text: combinedContent)

    if (gitDiffOutput.isEmpty()) {
        error("No changes found between commits.")
    }

    def approvalMail = """
        Hi Team, <br><br>
        The Build <b> ${env.BUILD_NUMBER} of ${env.JOB_NAME} has completed. </b> <br><br>
        <b> Commit ID: ${env.GIT_COMMIT} </b> <br><br>
        <b> Previous Commit ID: ${previousCommit} </b> <br><br>
        <b> Docker tag: ${env.DOCKER_TAG} </b> <br><br>
        <b> Source Path: ${env.WORKSPACE} </b> <br><br>
        <b> Author: ${authorEmail} </b> <br><br>
        <b> Date: ${env.BUILD_TIMESTAMP} </b> <br><br>
        <b> Build Result: ${buildStatus} </b> <br><br>
        Please review and approve or reject this build.
    """

    emailext (
        subject: mailSubject,
        body: approvalMail,
        mimeType: 'text/plain',
        to: 'thoshlearn@gmail.com',
        attachmentsPattern: 'changelog.txt'
    )
  timeout(time: 10, unit: 'MINUTES') {
                  input message: 'Waiting for Manager Approval'                }
}

def sendFailureEmail(buildStatus) {
    // Modify this function to send a failure email for the previous build
    //def previousBuildNumber = currentBuild.number - 1
    //def previousBuildUrl = env.BUILD_URL.replace(env.BUILD_NUMBER, previousBuildNumber.toString())
    def currentCommit = sh(returnStdout: true, script: 'git rev-parse HEAD')
    def previousCommit = sh(returnStdout: true, script: 'git rev-parse HEAD~1')
    def gitDiffOutput = sh(script: "git diff HEAD~1 ${currentCommit}", returnStdout: true)
    def changes = sh(script: 'git show --name-status HEAD^', returnStdout: true).trim()
    def mailSubject = "Failure Notification for Build - ${env.BUILD_NUMBER}"
    def combinedContent = changes + "\n\n" + gitDiffOutput
    writeFile(file: 'changelog.txt', text: combinedContent)
    def failureMail = """
        Build <b> ${env.JOB_NAME} #${env.BUILD_NUMBER} has failed. </b> <br><br>
        <b> Build Result: ${buildStatus} </b> <br><br>
        <b> Commit ID: ${env.GIT_COMMIT} </b> <br><br>
        <b> Previous Commit ID: ${previousCommit} </b> <br><br>
        <b> Source Path: ${env.WORKSPACE} </b> <br><br>
        <b> Author: ${authorEmail} </b> <br><br>
        <b> Date: ${env.BUILD_TIMESTAMP} </b> <br><br>
    """

    emailext (
        subject: mailSubject,
        body: failureMail,
        mimeType: 'text/plain',
        to: 'thoshlearn@gmail.com'
        attachmentsPattern: 'changelog.txt'
    )
}

