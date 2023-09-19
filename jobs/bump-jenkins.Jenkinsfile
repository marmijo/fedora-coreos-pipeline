import groovy.json.*
node {
    checkout scm
    // these are script global vars
    pipeutils = load("utils.groovy")
    pipecfg = pipeutils.load_pipecfg()
}

repo = "coreos/fedora-coreos-config"

properties([
    pipelineTriggers([]),
    parameters([
      choice(name: 'STREAM',
             choices: pipeutils.get_streams_choices(pipecfg),
             description: 'CoreOS stream to build'),
      string(name: 'VERSION',
             description: 'Override default versioning mechanism',
             defaultValue: '',
             trim: true),
      string(name: 'ADDITIONAL_ARCHES',
             description: "Override additional architectures (space-separated). " +
                          "Use 'none' to only build for x86_64. " +
                          "Supported: ${pipeutils.get_supported_additional_arches().join(' ')}",
             defaultValue: "",
             trim: true),
      booleanParam(name: 'FORCE',
                   defaultValue: false,
                   description: 'Whether to force a rebuild'),
      booleanParam(name: 'EARLY_ARCH_JOBS',
                   defaultValue: true,
                   description: "Fork off the multi-arch jobs before all tests have run"),
      booleanParam(name: 'ALLOW_KOLA_UPGRADE_FAILURE',
                   defaultValue: false,
                   description: "Don't error out if upgrade tests fail (temporary)"),
      booleanParam(name: 'CLOUD_REPLICATION',
                   defaultValue: false,
                   description: 'Force cloud image replication for non-production'),
      string(name: 'COREOS_ASSEMBLER_IMAGE',
             description: 'Override coreos-assembler image to use',
             defaultValue: "",
             trim: true),
      booleanParam(name: 'KOLA_RUN_SLEEP',
                   defaultValue: false,
                   description: 'Wait forever at kola tests stage. Implies NO_UPLOAD'),
      booleanParam(name: 'NO_UPLOAD',
                   defaultValue: false,
                   description: 'Do not upload results to S3; for debugging purposes.'),
      booleanParam(name: 'WAIT_FOR_RELEASE_JOB',
                   defaultValue: false,
                   description: 'Wait for the release job and propagate errors.'),
    ] + pipeutils.add_hotfix_parameters_if_supported()),
    buildDiscarder(logRotator(
        numToKeepStr: '100',
        artifactNumToKeepStr: '100'
    )),
    durabilityHint('PERFORMANCE_OPTIMIZED')
])

shwrap("git clone --branch bump-jenkins https://github.com/marmijo/fedora-coreos-pipeline.git")

def plugins_lockfile = "jenkins/controller/plugins.txt"
def plugins = shwrap("cat $plugins_lockfile | grep -v ^#")

plugins.each { plugin ->
    println ("plugin: ${plugin}")
}


// // Define the list of plugins and their versions
// def plugins = [
//     'basic-branch-build-strategies:1.3.2',
//     'generic-webhook-trigger:1.84.2',
//     'github-oauth:0.39',
//     'kubernetes-credentials-provider:1.199.v4a_1d1f5d074f',
//     'pipeline-github:2.8-138.d766e30bb08b',
//     'slack:625.va_eeb_b_168ffb_0',
//     'timestamper:1.20',
//     'splunk-devops-extend:1.9.9',
//     'splunk-devops:1.9.9',
//     'antisamy-markup-formatter:162.v0e6ec0fcfcf6',
//     'cloudbees-disk-usage-simple:182.v62ca_0c992a_f3'
// ]

// println "AR - plugins check below"
// // Check if each plugin's version is the latest
// plugins.each { plugin ->
//     def parts = plugin.split(':')
//     if (parts.size() == 2) {
//         def pluginName = parts[0]
//         def version = parts[1]
//         println ("pluginName:${pluginName}")
//         println ("version:${version}")
// }

// }

// println "test plugin url"

// def pluginUrl
// def pwd = env.WORKSPACE
// println ("PWD: $pwd")

// def pwd2

// node {
// pwd2 = shwrapCapture("pwd")

// }

// println ("PWD2: $pwd2")

// node {
// pluginUrl = shwrapCapture("curl -Ls -o /dev/null -w '%{url_effective}' https://updates.jenkins.io/download/plugins/cloudbees-disk-usage-simple/latest/cloudbees-disk-usage-simple.hpi")
// }



// //def pluginUrl = 'https://updates.jenkins.io/download/plugins/cloudbees-disk-usage-simple/latest/cloudbees-disk-usage-simple.hpi'

// // Extract the plugin version from the URL using a regular expression
// def versionPattern = /\/([^\/]+)\/([^\/]+)\/([^\/]+)\.hpi/
// def matcher = (pluginUrl =~ versionPattern)

// if (matcher.find()) {
//     def groupId = matcher.group(1)
//     def artifactId = matcher.group(2)
//     def pluginVersion = matcher.group(3)
//     println "Group ID: ${groupId}"
//     println "Artifact ID: ${artifactId}"
//     println "Plugin Version: ${pluginVersion}"
// } else {
//     println "Unable to extract plugin version from the URL."
// }


// println "AR plugin exit"
