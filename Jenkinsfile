pipeline {
    agent {
                dockerfile {
                    filename 'Dockerfile'
                    dir 'RFW'
                }
            }
    stages {
        stage('Back-end') {
            steps {
                sh 'ls -al'
                sh 'robot/robot.sh'
            }
        }
    }
}
