pipeline {
    agent any
    
     triggers {
        githubPush()
    }


    stages {
        stage('git ') {
            steps {
                git branch: 'master', url :'https://github.com/MedSlimane/devops-mohamed-slimane-4sim1'
            }
        }
        stage('compilation') {
            steps {
                sh 'mvn clean compile'
            }
        }
    }
}
