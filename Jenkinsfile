```/* SET THESE UP PER PROJECT */
projectName = "..."
gitlabPath = "..."

/* DO NOT TOUCH: REQUIRED FOR BUILD LOGIC */
dateTime = java.time.LocalDate.now()
environmentParts ="${env.JOB_BASE_NAME}".split("-")
environment = environmentParts[environmentParts.length - 1]

pipeline {
    agent none

    stages {
        /* Let's make sure we have the repository cloned to our workspace */
        stage('Clone repository') {
          steps {
                script {
                    node {
                        if (environment != "dev" && environment != "acc" && environment != "prod") {
                            error("Project name ends with " + environment + " but should end with 'dev', 'acc', or 'prod'. Example: 'build-${projectName}-dev'.")
                        }

                        checkout scm
                    }
                }
            }
        }

        /* This builds the actual image; synonymous to
        * docker build on the command line */
        stage('Build image') {
            steps {
                script {
                    node {
                        tagName = "${Tagname}"

                        if (tagName == "") {
                            shortCommit = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()
                            tagName = "${dateTime}-${shortCommit}"
                        }

                        sh 'echo "..." | sudo docker login registry.gitlab.com --username ... --password-stdin'

                        try {
                            sh "sudo docker build -t registry.gitlab.com/${gitlabPath}:${tagName} -t registry.gitlab.com/${gitlabPath}:latest-${environment} ."
                        } catch (Exception e) {
                            now = new Date()
                            dateTime = now.format("dd-MM-yyyy HH:mm")
                            commitData = sh(returnStdout: true, script: "git log -n 1 --oneline")
                            commitHistory = sh(returnStdout: true, script: "git log --graph --oneline -n 5 --pretty=format:\"%ad %an %s\" --date=short")
                            additionalMessage = ""

                            if (Deploy == "Build & deploy") {
                                additionalMessage = ": deploy aborted"
                            }

                            discordSend description: "```Check the logs to see what went wrong (probably TS errors)``````Last 5 commits:\n\n${commitHistory}```",
                            footer: "${projectName}-${environment} - " + dateTime + " - " + "${commitData}",
                            link: env.BUILD_URL + "/console",
                            result: "UNSTABLE",
                            thumbnail: "https://i.imgur.com/zSBw3LG.png",
                            title: "Build failed" + additionalMessage,
                            webhookURL: "..."
                        }

                        sh "sudo docker push registry.gitlab.com/${gitlabPath}:${tagName}"
                        sh "sudo docker push registry.gitlab.com/${gitlabPath}:latest-${environment}"
                    }
                }
            }
        }

        /* If enabled, launch the deployment job */
        stage('Deploy image') {
            steps {
                script {
                    node {
                        if (Deploy == "Build & deploy") {
                            jobResponse = sh(
                                returnStdout: true,
                                script: "curl 'http://scrumble:...@.../job/deploy-${projectName}/buildWithParameters?token=...&Omgeving=${environment}&Tag=registry.gitlab.com/${gitlabPath}:${tagName}'"
                            )

                            if (jobResponse.contains("404")) {
                                error("Deploy job 'deploy-${projectName}' does not exist")
                            }
                        }
                    }
                }
            }
        }
    }
}
```
