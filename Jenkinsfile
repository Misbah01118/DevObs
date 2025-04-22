pipeline {
    agent any

    environment {
        ARTIFACTORY_CREDS = credentials('artifactory-creds')  // Jenkins credentials ID for Artifactory
        SSH_KEY = credentials('ec2-ssh-key')                  // Jenkins credentials ID for EC2 SSH Key
        EC2_USER = 'ubuntu'
        EC2_IP = '<YOUR_EC2_PUBLIC_IP>'
        ZIP_FILE = 'nextjs-app.tar.gz'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/Misbah01118/DevObs.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build Next.js App') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Package App') {
            steps {
                sh 'tar -czf $ZIP_FILE .next public package.json'
            }
        }

        stage('Upload to Artifactory') {
            steps {
                script {
                    def server = Artifactory.server('ArtifactoryServerID')
                    def uploadSpec = """{
                        "files": [{
                            "pattern": "$ZIP_FILE",
                            "target": "generic-local/nextjs/"
                        }]
                    }"""
                    server.upload(uploadSpec)
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sh '''
                    scp -i $SSH_KEY $ZIP_FILE $EC2_USER@$EC2_IP:/home/ubuntu/
                    ssh -i $SSH_KEY $EC2_USER@$EC2_IP '
                        pkill node || true
                        rm -rf next-app && mkdir next-app
                        tar -xzf $ZIP_FILE -C next-app
                        cd next-app
                        nohup npm run start > next-app.log 2>&1 &
                    '
                '''
            }
        }
    }
}


