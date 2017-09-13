
node {
    docker.image("maven:3.5.0").inside {
        stage("checkout") {
            checkout scm
        }

        def scLogFile = "sc.log"
        def scPidFile = "sc.pid"

        try {
            stage("start sc") {
                sh "curl https://saucelabs.com/downloads/sc-4.4.9-linux.tar.gz -o sc.tar.gz"
                sh "tar -xvzf sc.tar.gz"
                sh "sc-4.4.9-linux/bin/sc --user ${SAUCE_USERNAME} --api-key ${SAUCE_KEY} --tunnel-identifier ${TUNNEL_IDENTIFIER} -x ${SC_PUBLIC_ENDPOINT} --pidfile=${scPidFile} > ${scLogFile} 2>&1 &"
            }

            stage("wait for sc") {
                try {
                    timeout(time: 3, unit: 'MINUTES') {
                        sh "( tail -f -n0 ${scLogFile} & ) | grep -q 'Sauce Connect is up, you may start your tests'"
                    }
                } catch (Exception e) {
                    println("Failed to connect to SC. Contents of log file:")
                    sh "cat ${scLogFile}"
                    throw e
                }
            }

            stage("start test") {
                sh "mvn test"
            }
        } finally {
            def pid = readFile scPidFile
            sh "kill ${pid} | echo No 'sc' process found to kill"
        }
    }
}