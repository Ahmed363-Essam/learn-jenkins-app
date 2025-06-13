pipeline {
    agent any

    environment{
        NETFLY_SITE_ID = 'ea20ac44-be47-4fd2-a211-cabe26c60c1d'
        NETLIFY_AUTH_TOKEN = credentials('netfly-jenkins-token')
        CI_ENVIRONMENT_URL = "https://symphonious-stardust-563860.netlify.app/"
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
                        image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                        reuseNode true
                    }
                }

                steps {
                    sh '''
                        npm install serve
                        node_modules/.bin/serve -s build &
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
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install  netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    
                    node_modules/.bin/netlify deploy --dir=build --auth=$NETLIFY_AUTH_TOKEN --site=$NETFLY_SITE_ID
                '''
            }
        }

        stage('Deply prod') {

            steps{
                input cancel: 'No , i cannot', message: 'DO you eant to deploy it to production ..............?', ok: 'yes , i am sure'
            }
        }

        stage('Deply') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install  netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    
                    node_modules/.bin/netlify deploy --dir=build --prod --auth=$NETLIFY_AUTH_TOKEN --site=$NETFLY_SITE_ID
                '''
            }
        }



            stage('Prod E2E') {
                agent {
                    docker {
                        image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                        reuseNode true
                    }
                }

            environment{
                CI_ENVIRONMENT_URL = "https://symphonious-stardust-563860.netlify.app/"
            }

                steps {
                    sh '''
                        npx playwright test --reporter=html
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
