stage('Check Commit') {
    steps {
        script {
            def msg = sh(
                script: "git log -1 --pretty=%B",
                returnStdout: true
            ).trim()

            if (msg.contains('[skip ci]')) {
                currentBuild.result = 'NOT_BUILT'
                error('Skipping build')
            }
        }
    }
}
