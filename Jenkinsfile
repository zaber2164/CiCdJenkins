pipeline {
    agent { label 'Label' }
    environment {
        DOTNET_VERSION = '6.0'
        SOLUTION_FILE = 'CiCdJenkins.sln' // Replace with the solution file name
        IIS_SERVER = '192.168.97.22' // Replace with your IIS server IP or hostname
        SITE_NAME = 'CiCdJenkins' // Replace with your IIS Site name
        IIS_USERNAME = credentials('webdeploy_user_username') // User for IIS (with web deploy permissions)
        IIS_PASSWORD = credentials('webdeploy_user_password') // Jenkins credentials for IIS
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
				withCredentials([usernamePassword(credentialsId: 'webdeploy_user', usernameVariable: 'IIS_USERNAME', passwordVariable: 'IIS_PASSWORD')]) {
					script {
						def deployCmd = "\"C:\\Program Files\\IIS\\Microsoft Web Deploy V3\\msdeploy.exe\" " +
										"-verb:sync " +
										"-source:contentPath='${WORKSPACE}\\publish' " +
										"-dest:contentPath='${SITE_NAME}',computerName='https://${IIS_SERVER}:8172/msdeploy.axd?site=${SITE_NAME}'," +
										"username=%IIS_USERNAME%,password=%IIS_PASSWORD%,authType='Basic' " +
										"-allowUntrusted"

						// Execute command safely without interpolating secrets in Groovy string
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
