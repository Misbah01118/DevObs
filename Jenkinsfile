pipeline {
  agent any

  environment {
    SSH_KEY = credentials('ec2-ssh-key')
    EC2_USER = 'ubuntu'
    EC2_IP = '3.108.42.98'
    ZIP_FILE = 'nextjs-app.tar.gz'
    MAVEN_OPTS = "-Dmaven.repo.local=.m2/repository"
  }

  stages {

    stage('Clone Repository') {
      steps {
        git branch: 'main', url: 'https://github.com/Misbah01118/DevObs.git'
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

    stage('Deploy to EC2') {
      steps {
        sh '''
          scp -i $SSH_KEY $ZIP_FILE $EC2_USER@$EC2_IP:/home/ubuntu/
          ssh -i $SSH_KEY $EC2_USER@$EC2_IP "pkill node || true && rm -rf next-app && mkdir next-app && tar -xzf nextjs-app.tar.gz -C next-app && cd next-app && nohup npm run start > next-app.log 2>&1 &"
        '''
      }
    }

    stage('Build and Deploy to Nexus') {
      steps {
        sh 'mvn clean deploy -DskipTests=true'
      }
    }

  }
}
