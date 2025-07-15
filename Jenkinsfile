def PROJECTS = []
pipeline {
    agent none
    
    environment {
        DOTNET_CLI_TELEMETRY_OPTOUT = '1'
        SOLUTION_FILE = 'JenkinsApi.sln'  // Update with your solution name
        DOCKER_REGISTRY = '-'  // Update with your registry
        BUILD_CONFIG = 'Release'
        CONFIG_FILE = 'docker-build.config'  // Config file name next to Dockerfile
        
    }
    
    stages {
        stage('Checkout') {
            agent any
            steps {
                checkout scm
            }
        }
        
        stage('Discover Projects') {
            agent any
            steps {
                script {
                    // Find all Dockerfiles and their config files
                    def dockerProjects = []
                    
                    findFiles(glob: '**/Dockerfile').each { dockerFile ->
                        def projectDir = dockerFile.path.split('/Dockerfile')[0]
                        def configFile = "${projectDir}/${CONFIG_FILE}"
                        
                        println projectDir
                        println configFile

                        if (fileExists(configFile)) {
                            // Read image name from config file
                            def config = readProperties file: configFile
                            dockerProjects << [
                                dir: projectDir,
                                image: config.IMAGE_NAME ?: "${projectDir.split('/')[-1]}",
                                context: config.BUILD_CONTEXT ?: projectDir
                            ]
                        } else {
                            // Fallback to directory name
                           println "Error: config file not found in${projectDir}"
                        }
                    }
                    
                    PROJECTS = dockerProjects
                    echo "Discovered projects: ${PROJECTS}"
                }
            }
        }
        
    //     stage('Build Docker Images') {
    //         parallel {
    //             script {
    //                 PROJECTS.each { project ->
    //                     stage("Build ${project.image}") {
    //                         agent {
    //                             label 'docker-agent'
    //                         }
    //                         steps {
    //                             script {
    //                                 dir(project.context) {
    //                                     // Build .NET project
    //                                     bat "dotnet build --configuration ${BUILD_CONFIG} --no-restore"
                                        
    //                                     // Read version from project (optional)
    //                                     def version = sh(
    //                                         script: 'dotnet gitversion /showvariable SemVer',
    //                                         returnStdout: true
    //                                     ).trim()
                                        
    //                                     // Build Docker image
    //                                     def fullImageName = "${DOCKER_REGISTRY}/${project.image}:${version}-${env.BUILD_NUMBER}"
    //                                     bat "docker build -t ${fullImageName} -f ${project.dir}/Dockerfile ."
                                        
    //                                     // // Push to registry
    //                                     // bat "docker push ${fullImageName}"
                                        
    //                                     // Store image reference for deployment
    //                                     env["IMAGE_${project.image.toUpperCase()}"] = fullImageName
    //                                 }
    //                             }
    //                         }
    //                     }
    //                 }
    //             }
    //         }
    //     }
    }
    
    // post {
    //     always {
    //         script {
    //             // Print all built images
    //             env.PROJECTS.each { project ->
    //                 def envVar = "IMAGE_${project.image.toUpperCase()}"
    //                 echo "${project.image}: ${env[envVar]}"
    //             }
    //         }
    //         cleanWs()
    //     }
    // }
}
