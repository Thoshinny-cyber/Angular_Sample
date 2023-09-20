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
post{
    always{
        def changeLog = checkout(
            poll: false,
            scm: [$class: 'GitSCM', branches: [[name: env.BRANCH_NAME]]]
    )
    
    // Send an email notification to the manager with the changelog as an attachment
    emailext (
        subject: "Approval Required for Build - ${currentBuild.displayName}",
        body: """
        Commit ID: ${env.GIT_COMMIT}
        Source Path: ${env.WORKSPACE}
        Author: ${env.BUILD_USER}
        Date: ${env.BUILD_TIMESTAMP}
        Branch: ${env.BRANCH_NAME}
        Build Status: ${currentBuild.result}

        Please review and approve or reject this build.
        To approve, reply to this email with 'APPROVE' in the subject.
        To reject, reply to this email with 'REJECT' in the subject.
        """,
        mimeType: 'text/plain',
        to: 'thoshlearn@gmail.com', // Manager's email address
        attachmentsPattern: "${changeLog}/changelog.txt", // Attach the changelog as an text file
        attachBuildLog: true // Attach the build log
    )

    // Wait for manager approval
    try {
        input message: 'Waiting for Manager Approval', submitter: 'thoshlearn@gmail.com'
    } catch (org.jenkinsci.plugins.workflow.steps.FlowInterruptedException e) {
        // Handle approval or rejection
        if (e.causes.any { it instanceof hudson.model.Cause$UserIdCause }) {
            echo "Build approved by manager"

            // Proceed to the next stage after approval
            stage('Docker Build'){
    steps{
        sh "echo ${DOCKER_TAG}"
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
                        execTimeout: 200000000,
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

            
        } else {
            error "Build rejected by manager"
        }
    }
    }
}
 

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
