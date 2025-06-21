pipeline {
    agent any

    environment{
        NETFLY_SITE_ID = 'ea20ac44-be47-4fd2-a211-cabe26c60c1d'
        NETLIFY_AUTH_TOKEN = credentials('netfly-jenkins-token')

    }

    stages {

        //         stage('Docker') {
        //     steps {
        //         sh 'docker build -t my-playwright .'
        //     }
        // }


        
        stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                }
            }
            steps {
                sh '''
                    aws --version
                '''
            }
        }
        



        

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {

            script{
                env.MY_VAR = 'this is my var'
            }


                sh '''
              
                    ls -la
                    node --version
                    npm --version
                    npm config set registry https://registry.npmmirror.com
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        


        stage('TESTING STAGE'){
            parallel{

            stage('Test') {
                agent {
                    docker {
                        image 'node:18-alpine'
                        reuseNode true
                    }
                }

                steps {
                    sh '''
                    echo "MY_VAR is $MY_VAR"
                        test -f build/index.html
                        npm test
                    '''
                }

                                    post {
                    always {

                        junit 'jest-results/junit.xml'                        }
                    }
            }
/* FOR LOCAL */
            stage('E2E') {
                agent {
                    docker {
                        image 'my-playwright'
                        reuseNode true
                    }
                }

                steps {
                    sh '''

                        serve -s build &
                        sleep 10
                        npx playwright test --reporter=html
                    '''
                }

                    post {
                    always {

                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright-report HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
            }




            }
        }


        stage('Deply dev') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            steps {
                sh '''

                    netlify --version
                    netlify deploy --dir=build --auth=$NETLIFY_AUTH_TOKEN --site=$NETFLY_SITE_ID
                '''
            }
        }



        stage('Deply') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            steps {
                sh '''
             
                                netlify --version
                    echo "Deploying to staging. Site ID: $NETFLY_SITE_ID"
       
                    netlify deploy --dir=build --prod --auth=$NETLIFY_AUTH_TOKEN --site=$NETFLY_SITE_ID
  
                

                    
                    
                '''
            }
        }


            stage('Prod E2E') {
                agent {
                    docker {
                        image 'my-playwright'
                        reuseNode true
                    }
                }

            environment{
                CI_ENVIRONMENT_URL = "https://symphonious-stardust-563860.netlify.app/"
            }

                steps {
                    sh '''

                     node --version
                    
                    netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
               
                    netlify deploy --dir=build --prod --auth=$NETLIFY_AUTH_TOKEN --site=$NETFLY_SITE_ID
                    npx playwright test  --reporter=html

    
                    '''
                }

                    post {
                    always {

                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'E2E PROD HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
            }




    }




}
