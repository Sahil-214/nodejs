// pipeline {
//     agent any
//     stages {
//         stage('Checkout') {
//             steps {
//                 // Check out code from GitHub
//                 checkout scm
//             }
//         }
//         stage('Run Ansible Playbook') {
//             steps {
//                 withCredentials([sshUserPrivateKey(
//                     credentialsId: 'jenkins',
//                     keyFileVariable: 'SSH_KEY'
//                 )]) {
//                     sh '''
//                         ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory.ini playbook.yml \
//                           -u ubuntu --private-key $SSH_KEY
//                     '''
//                 }
//             }
//         }
//     }
//     post {
//         success {
//             echo 'Pipeline completed successfully.'
//         }
//         failure {
//             echo 'Pipeline failed.'
//         }
//     }
// }

pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'docker.io'
        APP_NAME = 'nodejs'
        DOCKER_USERNAME = credentials('docker-hub-credentials') // Uses Jenkins credentials
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Version Bump') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-token', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                        def version = sh(script: "git describe --tags --abbrev=0", returnStdout: true).trim()
                        def (major, minor, patch) = version.tokenize('.')
                        patch = patch.toInteger() + 1
                        newVersion = "${major}.${minor}.${patch}"

                        echo "New Version: ${newVersion}"
                        sh "echo '${newVersion}' > VERSION"
                        sh "git config --global user.email 'mohammedarsalan204@gmail.com'"
                        sh "git config --global user.name 'ItsArsalanMD'"

                        sh "git remote set-url origin https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/ItsArsalanMD/nodejs.git"
                        sh "git add VERSION"
                        sh "git commit -m 'Version bump to ${newVersion} [ci skip]'"
                        sh "git checkout main"
                        sh "git pull origin main"
                        sh "git push origin main"
                    }
                }
            }
        }

        stage('Authenticate Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME_USR', passwordVariable: 'DOCKER_USERNAME_PSW')]) {
                        echo "Docker Username: ${DOCKER_USERNAME_USR}"  // Debugging Docker Username
                        sh "echo ${DOCKER_USERNAME_PSW} | docker login -u ${DOCKER_USERNAME_USR} --password-stdin"
                    }
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    def imageTag = "${DOCKER_REGISTRY}/${DOCKER_USERNAME_USR}/${APP_NAME}:${newVersion}"
                    def latestTag = "${DOCKER_REGISTRY}/${DOCKER_USERNAME_USR}/${APP_NAME}:latest"

                    echo "Building Docker Image: ${imageTag}"  // Debugging the image tag
                    echo "Tagging Latest: ${latestTag}"

                    sh "docker build -t ${imageTag} ."  // Build image with version
                    sh "docker tag ${imageTag} ${latestTag}"  // Tag with latest
                    sh "docker push ${imageTag}"  // Push versioned image
                    sh "docker push ${latestTag}"  // Push latest image
                }
            }
        }
    }
}

