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
        DOCKER_REGISTRY = 'docker.io/arsalan94'
        APP_NAME = 'nodejs'
        DOCKER_USERNAME = 'arsalan94'
        DOCKER_PASSWORD = 'Arsalan@1234$'
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
                    def version = sh(script: "git describe --tags --abbrev=0", returnStdout: true).trim()
                    def (major, minor, patch) = version.tokenize('.')
                    patch = patch.toInteger() + 1
                    def newVersion = "${major}.${minor}.${patch}"
                    
                    echo "New Version: ${newVersion}"
                    sh "echo '${newVersion}' > VERSION"
                    sh "git config --global user.email 'jenkins@example.com'"
                    sh "git config --global user.name 'Jenkins'"
                    sh "git add VERSION"
                    sh "git commit -m 'Version bump to ${newVersion}'"
                    sh "git push origin main"
                }
            }
        }
        
        stage('Authenticate Docker Hub') {
            steps {
                script {
                    sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin"
                }
            }
        }
        
        stage('Build and Push Docker Image') {
            steps {
                script {
                    def imageTag = "${DOCKER_REGISTRY}/${APP_NAME}:${newVersion}"
                    def latestTag = "${DOCKER_REGISTRY}/${APP_NAME}:latest"
                    
                    sh "docker build -t ${imageTag} ."
                    sh "docker tag ${imageTag} ${latestTag}"
                    sh "docker push ${imageTag}"
                    sh "docker push ${latestTag}"
                }
            }
        }
    }
}

