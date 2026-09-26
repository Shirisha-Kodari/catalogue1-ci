pipeline {

    agent {
        label 'roboshop'
    }

    environment {
        def appVersion = "1.0.0"
        acc_id = "936819548867"
        project = "roboshop"
        component = "catalogue1"
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

        stage('Read package.json') {  //read jenins appvesrion from packages .json
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    // Extract the version property
                    appVersion = packageJson.version
                    echo "The application version is: ${appVersion}" //print app vesrion 
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
                script {
                    // in this block we get aws authentication
                    withAWS(credentials: 'aws-credns', region: 'us-east-1') { //install pluin aws steps and retrive credns  
                        sh """
                            aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${acc_id}.dkr.ecr.us-east-1.amazonaws.com
                            docker build -t ${acc_id}.dkr.ecr.us-east-1.amazonaws.com/${project}/${component}:${appVersion} .
                        """
                    }
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
                                936819548867.dkr.ecr.us-east-1.amazonaws.com/${project}/${component}:${appVersion} .
                                '''
                                ,
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



