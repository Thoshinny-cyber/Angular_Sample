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
        stage('Approval') {
            steps {
                // Send an email notification to the manager for approval
               script{
                 def approvalMail = """
                    Build ${env.BUILD_NUMBER} of ${env.JOB_NAME} has completed.
                    SCM revision: ${env.GIT_COMMIT}
                    Docker tag: ${env.DOCKER_TAG}

                    Please review and approve or reject this build.
                    To approve, reply to this email with 'APPROVE' in the subject.
                    To reject, reply to this email with 'REJECT' in the subject.
                    """
                 def mailSubject =  "Approval Required for Build - ${currentBuild.displayName}"
                
                emailext (
                    subject: mailSubject,
                    body: approvalMail,
                    mimeType: 'text/plain',
                    to: 'thoshlearn@gmail.com', // Manager's email address
                    //attachmentsPattern: "${currentBuild.changeSets.fileChanges.file}", // Attach the changelog as a text file
                    attachLog: true, // Attach the build log
                    replyTo: currentBuild.upstreamBuilds[0]?.actions.find { it instanceof hudson.model.CauseAction }?.cause.upstreamProject
                )

                // Wait for manager approval
                timeout(time: 1, unit: 'HOURS') {
                        def approvalResponse = waitForEmailApproval(mailSubject)
                        if (approvalResponse == 'APPROVE') {
                            echo "Build approved by manager"
                            // Proceed to the next stage after approval
                        } else {
                            error "Build rejected by manager"
                        }
                    }
            }
        }
    }

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
            }
            }
        //}
    //}

          

//         stage('Docker Build'){
//     steps{
//         sh "echo ${DOCKER_TAG}"
//         sshPublisher(publishers: [
//             sshPublisherDesc(
//                 configName: 'dockerhost',
//                 transfers: [
//                     sshTransfer(
//                         cleanRemote: false,
//                         excludes: '',
//                         execCommand: """cd /opt/docker1; 
//                                         tar -xf Angular.tar.gz; 
//                                         docker build . -t thoshinny/angularapp:${DOCKER_TAG}
//                                         docker push thoshinny/angularapp:${DOCKER_TAG}
//                                         """,
//                         execTimeout: 200000,
//                         flatten: false,
//                         makeEmptyDirs: false,
//                         noDefaultExcludes: false,
//                         patternSeparator: '[, ]+$',
//                         remoteDirectory: '//opt//docker1',
//                         remoteDirectorySDF: false,
//                         removePrefix: '',
//                         sourceFiles: '**/*.gz'
//                     )
//                 ],
//                 usePromotionTimestamp: false,
//                 useWorkspaceInPromotion: false,
//                 verbose: true
//             )
//         ])
//     }
// }

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

// Function to wait for manager's approval email
def waitForEmailApproval(mailSubject) {
    def approvalTimeout = 60 * 60  
    def startTime = System.currentTimeMillis()
    while (System.currentTimeMillis() - startTime < approvalTimeout) {
        def mail = emailextFindLastMail(subject: mailSubject)
        if (mail) {
            def emailBody = mail.getContentType() == 'text/html' ? mail.getContent().toString() : mail.getContent().text
            if (emailBody.contains('APPROVE')) {
                return 'APPROVE'
            } else if (emailBody.contains('REJECT')) {
                return 'REJECT'
            }
        }
        sleep(60000) // Sleep for 1 minute before checking again
    }
    return 'TIMEOUT'
}
