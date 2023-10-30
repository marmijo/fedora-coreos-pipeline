import groovy.json.*
node {
    checkout scm
    // these are script global vars
    pipeutils = load("utils.groovy")
    pipecfg = pipeutils.load_pipecfg()
}

properties([
    pipelineTriggers([
        // check once a month
        pollSCM('H H 1 * *')
    ]),
    buildDiscarder(logRotator(
        numToKeepStr: '100',
        artifactNumToKeepStr: '100'
    )),
    durabilityHint('PERFORMANCE_OPTIMIZED')
])

botCreds = "github-coreosbot-releng-token-username-password"
//coreosbot_token = "github-coreosbot-releng-token-text"
pr_branch = "jp"

def getPluginLatestVersion(pluginsUpdate) {
    // Extract the plugin version from the URL 
    def versionPattern = /\/([^\/]+)\/([^\/]+)\/([^\/]+)\.hpi/
    def matcher = (pluginsUpdate =~ versionPattern)
    def pluginVersion

    if (matcher.find()) {
        def groupId = matcher.group(1)
        pluginVersion = matcher.group(2)
    } else {
        println "Unable to extract plugin version from the URL."
    }
    return pluginVersion
}

lock(resource: "bump-jenkins") {
    node{
        try {
            shwrap("""
                git config --global user.name "CoreOS Bot Releng"
                git config --global user.email "coreosbot-releng@fedoraproject.org"
            """)

            def pluginslist
            def pluginsToUpdate = [:]
            def haveChanges=false 
            def repo = "coreos/fedora-coreos-pipeline"

            stage("Read plugins.txt") {
                shwrapCapture("""
                    if [[ -d fedora-coreos-pipeline ]]; then
                        rm -rf fedora-coreos-pipeline
                    fi
                    git clone --branch jp https://github.com/aaradhak/fedora-coreos-pipeline.git
                """)
                def plugins_lockfile = "jenkins/controller/plugins.txt"
                pluginslist = shwrapCapture("cat $plugins_lockfile | grep -v ^#").split('\n')
            }

            stage("Check for plugin updates") {
                def pluginsUpdate
                pluginslist.each { plugin ->
                    def parts = plugin.split(':')
                    if (parts.size() == 2) {
                        def pluginName = parts[0]
                        def currentVersion = parts[1]
                        pluginsUpdate = shwrapCapture("curl -Ls -o /dev/null -w '%{url_effective}' https://updates.jenkins.io/download/plugins/${pluginName}/latest/${pluginName}.hpi")
                        def latestVersion = getPluginLatestVersion(pluginsUpdate)
                        if (latestVersion.toString() != currentVersion.toString()) {
                            haveChanges = true
                            pluginsToUpdate["${pluginName}"] = [currentVersion, latestVersion]
                            println("Plugin: ${pluginName} current version is ${currentVersion}, it will be updated to latest version: ${latestVersion}")
                            shwrap("""
                                sed -i 's/${pluginName}:${currentVersion}/${pluginName}:${latestVersion}/g' jenkins/controller/plugins.txt
                            """)
                        }
                        else {
                            println("The latest version of ${pluginName} is already installed: ${currentVersion}")
                        }
                    }
                    else {
                        println("ERROR: unexpected")
                    }
                }
            }
        
            stage("Push") {
                if (haveChanges){
                    def message = "bump jenkins plugin version"
                    shwrap("git add jenkins/controller/plugins.txt")
                    shwrap("git commit -m '${message}' -m 'Job URL: ${env.BUILD_URL}' -m 'Job definition: https://github.com/coreos/fedora-coreos-pipeline/blob/main/jobs/bump-jenkins-plugins.Jenkinsfile'")
                    withCredentials([usernamePassword(credentialsId: botCreds,
                                                      usernameVariable: 'GHUSER',
                                                      passwordVariable: 'GHTOKEN')]) {
                    
                        curl -H "Authorization: token ${GHTOKEN}" \
                        -X POST \
                        -d '{ \
                            "title": "Bump jenkins plugin version to the latest", \
                            "head": "${pr_branch}", \
                            "base": "main" \
                        }' \ 
                        https://api.github.com/repos/marmijo/fedora-coreos-pipeline/pulls
                    }
                }
            }
/*
            stage ("Create a Pull Request"){
                curl -H "Authorization: token ${coreosbot_token}" \
                -X POST \
                -d '{
                    "title": "Bump jenkins plugin version to the latest",
                    "head": "${pr_branch}",
                    "base": "main"
                }' \ 
                https://api.github.com/repos/marmijo/fedora-coreos-pipeline/pulls
            }
*/
        } catch (e) {
            currentBuild.result = 'FAILURE'
            throw e
        } finally {
            if (currentBuild.result != 'SUCCESS') {
                pipeutils.trySlackSend(message: "bump-lockfile #${env.BUILD_NUMBER} <${env.BUILD_URL}|:jenkins:> <${env.RUN_DISPLAY_URL}|:ocean:> [${params.STREAM}]")
            }
        }
    }
}
