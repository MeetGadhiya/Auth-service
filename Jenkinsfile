#!/usr/bin/env groovy

pipeline {

    agent any

    environment {
        SERVICE_NAME = 'Reg-Webapp'
        DIST_DIR = 'dist'
        ARTIFACT_NAME = "${env.SERVICE_NAME}-${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
    }

    parameters {
        choice(name: 'DEPLOY_ENV', choices: 'dev\ndev2\nqa\nstaging\nprod', description: 'Select the deployment environment')
        choice(name: 'PRIORITY', choices:'3\n2\n1', description: 'Select job priority')
    }

    stages {

        stage('Install') {

            steps {
                print "Branch name: ${env.BRANCH_NAME}"
                sh 'node -v'
                sh 'npm -v'
                sh 'npm install'
            }

        }

        stage("Set DEPLOY_ENV "){

             when {
                environment name: 'DEPLOY_ENV' , value: null
            }

            steps{
                script {
                    env.DEPLOY_ENV = 'dev'
                }
             }
        }

        stage('Build') {

            steps {
                sh "npm run build"
            }

        }

        stage('Package') {

            steps {
                dir("${env.DIST_DIR}") {
                    sh "tar -czf ${env.WORKSPACE}/${env.ARTIFACT_NAME}.tar.gz ."
                }
            }

        }

        stage('Deploy - Dev') {

            when {
                environment name: 'DEPLOY_ENV', value: 'dev'
            }

            steps {
                sh "ssh ubuntu@10.0.5.187 'bash -s' < ./pre-deploy.sh ${env.SERVICE_NAME}"
                sh "scp ${env.WORKSPACE}/${env.ARTIFACT_NAME}.tar.gz ubuntu@10.0.5.187:/home/ubuntu/.tmp/builds/${env.SERVICE_NAME}"
                sh "ssh ubuntu@10.0.5.187 'bash -s' < ./deploy.sh ${env.SERVICE_NAME} ${env.ARTIFACT_NAME} ${env.DEPLOY_ENV}"
            }

        }

    }

}
