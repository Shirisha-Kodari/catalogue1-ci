// pipeline {

//     agent {
//         label 'roboshop'
//     }

//     environment {
//         def appVersion = ""
//         acc_id = "936819548867"
//         project = "roboshop"
//         component = "catalogue1"
//     }

//     options {
//         timeout(time: 30, unit: 'MINUTES')
//         disableConcurrentBuilds()
//     }

//     parameters {
//         booleanParam(
//             name: 'deploy',
//             defaultValue: false,
//             description: 'Deploy application'
//         )
//     }

//     stages {

//         stage('Read package.json') {  //read jenins appvesrion from packages .json
//             steps {
//                 script {
//                     def packageJson = readJSON file: 'package.json'
//                     // Extract the version property
//                     appVersion = packageJson.version
//                     echo "The application version is: ${appVersion}" //print app vesrion 
//                 }
//             }
//         }

//         stage('Install Dependencies') {
//             steps {
//                 sh '''
//                     npm install
//                 '''
//             }
//         }

//         stage('Unit Testing') {
//             steps {
//                 sh '''
//                     echo "Running unit tests"
//                 '''
//             }
//         }

//         stage('Docker Build') {
//             steps {
//                 script {
//                     // in this block we get aws authentication
//                     withAWS(credentials: 'aws-credns', region: 'us-east-1') { //install pluin aws steps and retrive credns  
//                         sh """
//                             aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${acc_id}.dkr.ecr.us-east-1.amazonaws.com
//                             docker build -t ${acc_id}.dkr.ecr.us-east-1.amazonaws.com/${project}/${component}:${appVersion} .
//                         """
//                     }
//                 }
//             }
//         }
//     }
//        stage('Trivy Scan') {
//             steps {
//                 script {
//                     def dockerfileScan = sh(
//                         script: """
//                             trivy config --exit-code 1 --severity HIGH,CRITICAL --format table ./Dockerfile
//                         """,
//                         returnStatus: true
//                     )

//                     def imageScan = sh(
//                         script: """
//                             trivy image --scanners vuln --pkg-types os --exit-code 1 --severity HIGH,CRITICAL --format table ${acc_id}.dkr.ecr.us-east-1.amazonaws.com/${project}/${component}:${appVersion}
//                         """,
//                         returnStatus: true
//                     )

//                     if (dockerfileScan != 0 || imageScan != 0) {
//                         error "Trivy found HIGH/CRITICAL issues in Dockerfile and/or OS packages. Failing pipeline."
//                     }
//                 }
//             }
//         }
//     post {

//         always {
//             echo 'Cleaning workspace...'
//             deleteDir()
//         }

//         success {
//             echo 'Pipeline completed successfully!'
//         }

//         failure {
//             echo 'Pipeline failed!'
//         }
//     }
// }


pipeline {

    agent {
        label 'roboshop'
    }

    environment {
        APP_VERSION = ''
        ACC_ID = '936819548867'
        PROJECT = 'roboshop'
        COMPONENT = 'catalogue1'
        AWS_REGION = 'us-east-1'
        ECR_REGISTRY = '936819548867.dkr.ecr.us-east-1.amazonaws.com'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    parameters {
        booleanParam(
            name: 'deploy',
            defaultValue: false,
            description: 'Deploy application'
        )
    }

    stages {

        stage('Read package.json') {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'

                    env.APP_VERSION = packageJson.version

                    echo "The application version is: ${env.APP_VERSION}"
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    npm install
                '''
            }
        }

        stage('Unit Testing') {
            steps {
                sh '''
                    echo "Running unit tests"
                '''
            }
        }

        stage('Docker Build') {
            steps {
                withAWS(
                    credentials: 'aws-credns',
                    region: 'us-east-1'
                ) {
                    sh '''
                        echo "Logging into AWS ECR..."

                        aws ecr get-login-password --region $AWS_REGION |
                        docker login \
                            --username AWS \
                            --password-stdin $ECR_REGISTRY

                        echo "Building Docker image..."

                        docker build \
                            -t $ECR_REGISTRY/$PROJECT/$COMPONENT:$APP_VERSION .
                    '''
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                script {

                    echo "Scanning Dockerfile..."

                    def dockerfileScan = sh(
                        script: '''
                            trivy config \
                                --exit-code 1 \
                                --severity HIGH,CRITICAL \
                                --format table \
                                ./Dockerfile
                        ''',
                        returnStatus: true
                    )

                    echo "Scanning Docker image..."

                    def imageScan = sh(
                        script: '''
                            trivy image \
                                --scanners vuln \
                                --pkg-types os \
                                --exit-code 1 \
                                --severity HIGH,CRITICAL \
                                --format table \
                                $ECR_REGISTRY/$PROJECT/$COMPONENT:$APP_VERSION
                        ''',
                        returnStatus: true
                    )

                    if (dockerfileScan != 0 || imageScan != 0) {
                        error "Trivy found HIGH/CRITICAL issues. Failing pipeline."
                    }

                    echo "Trivy scan completed successfully!"
                }
            }
        }
    }

    post {

        always {
            echo 'Cleaning workspace...'
            deleteDir()
        }

        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed!'
        }
    }
}