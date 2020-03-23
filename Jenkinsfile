pipeline {
    agent {
        label "master"
    }

    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "127.0.0.1:8081"
        NEXUS_REPOSITORY = "automat-it"
        NEXUS_CREDENTIAL_ID = "nexus-credentials"
    }
    stages {
        stage("Clone Source Code") {
            steps {
                script {
                    git 'https://github.com/zivkashtan/course.git';
                }
            }
        }
        stage("MVN Build") {
            steps {
                script {
                    sh "mvn package -DskipTests=true"
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                withSonarQubeEnv(credentialsId: 'fd4438b8-7777-4312-92e3-8e216643245d', installationName: 'SonarQube') {
                sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.6.0.1398:sonar'
                    }
                }
            }
        }

        stage("Publish to Nexus") {
            steps {
                script {
                    // Read POM xml file using 'readMavenPom' step
                    pom = readMavenPom file: "pom.xml";
                    // Find built artifact under target folder
                    filesByGlob = findFiles(glob: "web/target/*.war");
                    // Print some info from the artifact found
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    // Extract the path from the File found
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: "war"]
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'git clone https://github.com/shmuel-mor/automat-it.git'
                sh 'cp web/target/*.war automat-it/'
                sh 'cd automat-it; docker build -t time-tracker .'
                sh 'docker tag time-tracker localhost:8083/repository/docker-repo/time-tracker'
            }
        }
        stage('Push Image to Nexus') {
            steps {
                sh 'docker login -u admin -p admin http://localhost:8083/repository/docker-private/'
                sh 'docker push localhost:8083/repository/docker-repo/time-tracker'
            }
        }
    }
}
