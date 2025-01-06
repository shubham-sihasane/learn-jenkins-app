pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        AWS_DEFAULT_REGION = 'ap-south-1'
        AWS_ECS_CLUSTER_NAME = 'JenkinsCluster'
        AWS_ECS_SERVICE_NAME = 'jenkinsAppService'
        AWS_ECS_TD = 'jenkinsApp'
    }

    stages {

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        steps('Docker Build'){
            steps {
                sh 'docker build -t jenkinsApp .'
            }
        }

        // stage('Deploy to AWS') {
        //     agent {
        //         docker {
        //             image 'amazon/aws-cli'
        //             reuseNode true
        //             args "-u root --entrypoint=''"
        //         }
        //     }

        //     steps {
        //         withCredentials([usernamePassword(credentialsId: 'AWS-Credentials', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
        //             sh '''
        //                 aws --version
        //                 yum install jq -y
        //                 LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition.json | jq '.taskDefinition.revision')
        //                 aws ecs update-service --cluster $AWS_ECS_CLUSTER_NAME --service $AWS_ECS_SERVICE_NAME --task-definition $AWS_ECS_TD:$LATEST_TD_REVISION
        //                 aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER_NAME --services $AWS_ECS_SERVICE_NAME
        //             '''
        //         }
        //     }
        // }
    }
}