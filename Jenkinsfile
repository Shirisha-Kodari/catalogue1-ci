
    pipeline {
        // agent any 
        agent {
            label 'roboshop'
        }
        environment {
            appVersion = ''
            REGION = "us-east-1"
            ACC_ID = "936819548867"
            PROJECT = "roboshop"
            COMPONENT = "catalogue"
            //COURSE = 'jenkins'
        }

        // }
        options { // pipeline expries 30 mint
            timeout(time: 30, unit: 'MINUTES')
            disableConcurrentBuilds() // not parallel to pipelines at a time so, one complted after another complted .
            // prevents multiple runs of this job at the same time (avoids conflicts like 2 builds pushing same Docker tag).
        }
        parameters {
            booleanParam(name: 'deploy', defaultValue: false, description: 'Toggle this value')
            
        }
    // build
    stages {
            stage('Read package.json') { //read version and dowload version print it
                steps {
                    script {
                    
                        def packageJSON = readJSON file: 'package.json' // def means define 
                        appVersion = packageJSON.version
                        echo "Package Version: ${appVersion }"

                    }
                }
            }
            stage('install dependencies') {
                steps {
                    script {
                        sh """
                            npm install
                        """

                    }
                }
            }
            stage('unit testing') {
                steps {
                    script {
                        sh """
                            echo "unit tests"
                        """

                    }
                }
            } 
            
                         
            
            
        // if size is 0 failed the build 
            
        }  
        post { 
            always { 
                echo 'I will always say Hello again!'
                deleteDir() // delete post build pipeline in workspace  
            }
            success { 
                echo 'hello success'
            }
            failure { 
                echo 'hello failure'
            }
        }
    }

    