pipeline {
    agent any
     environment {
        NEXUS_INSTANCE_ID = "nexus"
        NEXUS_REPOSITORY = "devops-usach-nexus"
    }
    stages {
        stage('compile') {
            steps {
                echo 'Source code compilation in progress.....'
                script {
                    if(isUnix()) {
                        echo 'Unix OS'
                        sh './mvnw clean compile -e'
                    } else {
                        echo 'Windows OS'
                        bat 'mvnw clean compile -e'
                    }
                }
                echo '.....Source code compilation completed'
            }
        }
        stage('test') {
            steps {
                echo 'Source code testing in progress.....'
                script {
                    if(isUnix()) {
                        echo 'Unix OS'
                        sh './mvnw clean test -e'
                    } else {
                        echo 'Windows OS'
                        bat 'mvnw clean test -e'
                    }
                }
                echo '.....Source code testing completed'
            }
        }
        stage('package') {
            steps {
                echo 'Source code packaging in progress.....'
                script {
                    if(isUnix()) {
                        echo 'Unix OS'
                        sh './mvnw clean package -e'
                    } else {
                        echo 'Windows OS'
                        bat 'mvnw clean package -e'
                    }
                }
                echo '.....Source code packaging completed'
            }
        }
        stage('sonar') {
            steps {
                echo 'Sonar scan in progress.....'
                withSonarQubeEnv(credentialsId: '22f7a5b8-3425-4d58-a9e9-2326e6749326', installationName: 'sonarqube') {
                    script {
                        if(isUnix()) {
                            echo 'Unix OS'
                                sh './mvnw clean verify sonar:sonar \
                                     -Dsonar.projectKey=example-maven2'
                        } else {
                            echo 'Windows OS'
                                bat 'mvnw clean verify sonar:sonar \
                                    -Dsonar.projectKey=example-maven2'
                        }
                        echo '.....Sonar scan completed'
                    }
                }
            }
        }
        stage("uploadNexus") {
            steps {
                echo 'Uploading to nexus in progress.....'
                script {
                    pom = readMavenPom file: "pom.xml";
                    files = findFiles(glob: "build/*.${pom.packaging}");
                    echo """${files[0].name}
                            ${files[0].path}
                            ${files[0].directory}
                            ${files[0].length}
                            ${files[0].lastModified}"""
                    artifactPath = files[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo """File: ${artifactPath},
                              group: ${pom.groupId},
                              packaging: ${pom.packaging},
                              version ${pom.version}"""
                        nexusPublisher(
                            nexusInstanceId: NEXUS_INSTANCE_ID,
                            nexusRepositoryId: NEXUS_REPOSITORY,
                            packages: [
                                [
                                    $class: 'MavenPackage',
                                    mavenAssetList: [
                                        [classifier: '',
                                        extension: '',
                                        filePath: artifactPath]],
                                    mavenCoordinate:
                                        [artifactId: pom.artifactId,
                                        groupId: pom.groupId,
                                        packaging: pom.packaging,
                                        version: pom.version]
                                 ]
                            ]
                        )
                    echo '.....Artifact Uploaded successfully'
                    } else {
                        error "File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
    }
 }