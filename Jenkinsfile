pipeline {
    agent { label 'Label' }
    environment {
        DOTNET_VERSION = '6.0'
        SOLUTION_FILE = 'CiCdJenkins.sln' // Replace with the solution file name
        IIS_SERVER = 'T-DEV-22' // Replace with your IIS server IP or hostname
        SITE_NAME = 'CiCdJenkins' // Replace with your IIS Site name
    }
 
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/zaber2164/CiCdJenkins.git' // Replace with your GitHub URL
            }
        }
 
        stage('Restore') {
            steps {
                script {
                    // Restores NuGet packages
                    bat "dotnet restore ${SOLUTION_FILE}"
                }
            }
        }
 
        stage('Build') {
            steps {
                script {
                    // Builds the solution in Release mode
                    bat "dotnet build ${SOLUTION_FILE} -c Release"
                }
            }
        }
 
        stage('Publish') {
            steps {
                script {
                    // Publishes the project to a folder
                    bat "dotnet publish ${SOLUTION_FILE} -c Release -o ${WORKSPACE}/publish"
                }
            }
        }
 
        stage('Deploy to IIS') {
            steps {
                // Use the 'withCredentials' block to securely retrieve the stored username and password
                withCredentials([usernamePassword(credentialsId: 'webdeploy_user', usernameVariable: 'IIS_USERNAME', passwordVariable: 'IIS_PASSWORD')]) {
                    script {
					echo "Deploying with Username: ${IIS_USERNAME}"
					echo "Deploying with Password: ${IIS_PASSWORD}"
                        // Using the environment variables for username and password securely
                        def deployCmd = "\"C:\\Program Files\\IIS\\Microsoft Web Deploy V3\\msdeploy.exe\" " +
                                        "-verb:sync " +
                                        "-source:contentPath='${WORKSPACE}\\publish' " +
                                        "-dest:contentPath='${SITE_NAME}',computerName='https://${IIS_SERVER}:8172/msdeploy.axd'," +
                                        "username=%IIS_USERNAME%,password=%IIS_PASSWORD%,authType='Basic' " +
                                        "-allowUntrusted -verbose"
										
                        // Execute the deployment command
                        bat label: 'Deploy to IIS', script: deployCmd
                    }
                }
            }
        }
 
    }
 
    post {
        always {
            node('Label') {
                cleanWs()
            }
        }
    }
 
}