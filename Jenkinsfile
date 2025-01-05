pipeline {
    agent any

    environment {
      NETLIFY_SITE_ID = 'a51c62c7-e627-43e5-8b78-553bcbcaf5b1'
      NETLIFY_AUTH_TOKEN = credentials('Netlify-Token')
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
  
        stage('Testing') {
          parallel {
            stage('Test') {
              agent {
                  docker {
                      image 'node:18-alpine'
                      reuseNode true
                  }
              }

              steps {
                  sh '''
                      #test -f build/index.html
                      npm test
                  '''
              }
              post {
                always {
                    junit 'jest-results/junit.xml'
                }
            }
          }

          stage('E2ETest') {
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
                      publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                  }
              }
            }
          }
        }
        
        stage('Stage Deployment') {
              agent {
                  docker {
                      image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                      reuseNode true
                  }
              }
              environment {
                 = "${env.STAGING_URL}"
              }
              steps {
                  sh '''
                      node --version
                       npm install netlify-cli node-jq
                        node_modules/.bin/netlify --version
                        echo "Deploying to Staging: SITE ID - $NETLIFY_SITE_ID"
                        node_modules/.bin/netlify status
                        CI_ENVIRONMENT_URL=${node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json}
                        node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                      npx playwright test --reporter=html
                  '''
              }
              post {
                  always {
                      publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright STAGE Report', reportTitles: '', useWrapperFileDirectly: true])
                  }
              }
        }

        stage('Prod Approval') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Do you wish to deploy to production??', ok: 'Yes, I am sure!'
                }
            }
        }

        stage('Prod Deployment') {
              agent {
                  docker {
                      image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                      reuseNode true
                  }
              }
              environment {
                CI_ENVIRONMENT_URL = "https://happyjenkins.netlify.app"
              }
              steps {
                  sh '''
                      node --version
                      npm install netlify-cli
                      node_modules/.bin/netlify --version
                      echo "Deploying to Production: SITE ID - $NETLIFY_SITE_ID"
                      node_modules/.bin/netlify status
                      node_modules/.bin/netlify deploy --dir=build --prod
                      sleep 10
                      npx playwright test --reporter=html
                  '''
              }
              post {
                  always {
                      publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright PROD Report', reportTitles: '', useWrapperFileDirectly: true])
                  }
              }
        }
    }
}
