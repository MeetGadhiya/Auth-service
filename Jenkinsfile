#!/usr/bin/env groovy

pipeline {

    agent any

    environment {
        SERVICE_NAME = 'Auth-service'
        SRC_DIR = 'src'
        ARTIFACT_NAME = "${env.SERVICE_NAME}-${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
    }

    parameters {
        choice(name: 'DEPLOY_ENV', choices: 'dev\nqa\nstaging\nprod', description: 'Select the deployment environment')
    }

    stages {

        stage('build') {

            steps {
                print "Branch name: ${env.BRANCH_NAME}"
                sh 'node -v'
                sh 'npm -v'
                dir("${env.SRC_DIR}") {
                    sh 'npm-cache install npm'
                }
            }

        }

        stage('Package') {

            steps {
                dir("${env.SRC_DIR}") {
                    sh "tar -czf ${env.WORKSPACE}/${env.ARTIFACT_NAME}.tar.gz ."
                }
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

        stage('Deploy - Dev') {

            when {
                environment name: 'DEPLOY_ENV', value: 'dev'
            }

            steps {
                sh "ssh ubuntu@172.31.26.24 'bash -s' < ./pre-deploy.sh ${env.SERVICE_NAME}"
                sh "scp ${env.WORKSPACE}/${env.ARTIFACT_NAME}.tar.gz ubuntu@172.31.26.24:/home/ubuntu/.tmp/builds/${env.SERVICE_NAME}"
                sh "ssh ubuntu@172.31.26.24 'bash -s' < ./deploy.sh ${env.SERVICE_NAME} ${env.ARTIFACT_NAME} ${env.DEPLOY_ENV}"
            }

        }

    }

}
