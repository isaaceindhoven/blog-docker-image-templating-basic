def nodeVersion

// Define the build function that returns a Docker build task
def build(def version) {
    def now = new Date()
    def datetime = now.format("yyyy-MM-dd'T'HH-mm-ss")

    // Tag with the date and "latest"
    def tags = [
        datetime,
        "latest"
    ]

    // Pull the latest version of the image before building to ensure we're using Docker layer caching
    def image = docker.build(version.image, "--pull -f ./${ version.file } .")

    // Push all tags
    tags.each { tag -> 
        image.push(tag)
    }
}

pipeline {
    agent any

    options {
        // We perform our own checkout steps
        skipDefaultCheckout(true)

        // Prevent builds from running concurrently
        disableConcurrentBuilds()

        // Support ANSI colors
        ansiColor('xterm')
    }

    triggers {
        cron('@midnight')
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    deleteDir()
                    checkout(scm)
                }
            }
        }

        stage('Node version') {
            steps {
                script {
                    // Read the Node.js version from the ".nvmrc" file
                    nodeVersion = readFile(".nvmrc")

                    // Remove the "v" prefix, if present
                    nodeVersion = nodeVersion.replaceAll('v', '')

                    // Remove any whitespace if present
                    nodeVersion = nodeVersion.replaceAll('\\s', '')
                }
            }
        }

        stage('Create Dockerfiles') {
            steps {
                script {
                    // Run the NPM script in a Node.js Docker container with the version specified in the .nvmrc file.
                    docker
                        .image("node:${ nodeVersion }")
                        .inside() {
                            sh('npm ci')
                            sh('npm start')
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    docker.withRegistry("<your Docker registry here>", "Registry credentials") {

                        // Read the needed image versions from the generated output.json file
                        def versions = readJSON(file: 'output/output.json')

                        versions.each { version -> 
                            build(version)
                        }
                    }
                }
            }
        }
    }
}
