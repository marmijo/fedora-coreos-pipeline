node {
    checkout scm
    // these are script global vars
    pipeutils = load("utils.groovy")
    pipecfg = pipeutils.load_pipecfg()
}

properties([
    pipelineTriggers([
        // run every 24h only for now
        cron("H H * * *")
    ]),
    buildDiscarder(logRotator(
        numToKeepStr: '100',
        artifactNumToKeepStr: '100'
    )),
    durabilityHint('PERFORMANCE_OPTIMIZED')
])

node {
    def mechanical_streams = pipeutils.streams_of_type(pipecfg, 'mechanical')
    def scheduled_streams = pipeutils.scheduled_streams(pipecfg, mechanical_streams)

    if (scheduled_streams) {
        // if any mechanical stream has the `scheduled: true` knob set, only run those streams
        scheduled_streams.each{
            echo "Triggering build for scheduled stream: ${it}"
            build job: 'build', wait: false, parameters: [
              string(name: 'STREAM', value: it),
              booleanParam(name: 'EARLY_ARCH_JOBS', value: false)
            ]
        }
    } else {
        // only trigger all mechanical FCOS streams. There is another mechanism to trigger
        // these streams for RHCOS. Since the brew section of the config is required for
        // RHCOS, use it to decide which product we're building.
        def is_rhcos = pipecfg.brew != null
        if (!is_rhcos) {
            mechanical_streams.each{
                echo "Triggering build for mechanical stream: ${it}"
                build job: 'build', wait: false, parameters: [
                  string(name: 'STREAM', value: it),
                  booleanParam(name: 'EARLY_ARCH_JOBS', value: false)
                ]
            }
        }
    }
}
