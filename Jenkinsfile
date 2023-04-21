/* impoort shared library */
@Library('jenkins-shared-library') _

pipeline {
    environment {
        IMAGE_NAME = "alpinehelloworld"
        IMAGE_TAG = "latest"
        STAGING = "environment-staging"
        PRODUCTION = "environment-production"
    }
    agent none 
    stages {
        stage ('Build image') {
            agent any
            steps {
                script {
                    sh '''
                        echo "Build image"
                        docker build -t mndiayepro97/$IMAGE_NAME:$IMAGE_TAG .
                        '''
                }
            }
        }

        stage ('Run container based on builded image') {
            agent any
            steps {
                script {
                    sh '''
                        echo "Run container based on builded image"
                        docker run --name $IMAGE_NAME -d -p 80:5000 --network jenkins_default -e PORT=5000 mndiayepro97/$IMAGE_NAME:$IMAGE_TAG
                        sleep 5s
                    '''
                }
            }
        }

        stage ('Test image') {
            agent any
            steps {
                script {
                    sh '''
                        echo "Test image"
                        curl http://$IMAGE_NAME:5000 |grep -q 'Hello world!'
                    '''
                }
            }
        }

        stage ('Clean container') {
            agent any
            steps {
                script {
                    sh '''
                        echo "Clean container"
                        docker stop $IMAGE_NAME
                        docker rm $IMAGE_NAME
                    '''
                }
            }
        }

        stage ('Push image in staging and deploy it') {
            when {
                expression { GIT_BRANCH == 'origin/master' }
            }
                agent any
                environment {
                    HEROKU_API_KEY = credentials('api_key_heroku')
                }
                steps {
                    script {
                        sh '''
                            echo "Push image in staging and deploy it"
                            heroku container:login
                            heroku create $STAGING || echo "project already exists"
                            heroku container:push -a $STAGING web
                            heroku container:release -a $STAGING web
                        '''
                    }
                }
        }

        stage ('Push image in production and deploy it') {
            when {
                expression { GIT_BRANCH == 'origin/master' }
            }
                agent any
                environment {
                    HEROKU_API_KEY = credentials('api_key_heroku')
                }
                steps {
                    script {
                        sh '''
                            echo "Push image in production and deploy it"
                            heroku container:login
                            heroku create $PRODUCTION || echo "project already exists"
                            heroku container:push -a $PRODUCTION web
                            heroku container:release -a $PRODUCTION web
                        '''
                    }
                }
        }
    }
    post {
       always {
        script {
            slackNotifier currentBuild.result
            }
        }  
    }
}
