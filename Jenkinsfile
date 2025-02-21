pipeline {
    agent any
    stages {
        stage('Checkout/Build/Test') {
            steps {
                // Checkout files.
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: 'master']],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [], submoduleCfg: [],
                    userRemoteConfigs: [[
                        name: 'github',
                        url: 'https://github.com/mmorejon/time-table.git'
                    ]]
                ])

                // Build and Test
                sh 'xcodebuild -scheme "TimeTable" -configuration "Debug" build test -destination "platform=iOS Simulator,name=iPhone 7,OS=12.2" -enableCodeCoverage YES | /usr/local/bin/xcpretty -r junit'

                // Publish test restults.
                step([$class: 'JUnitResultArchiver', allowEmptyResults: true, testResults: 'build/reports/junit.xml'])
            }
        }

        stage('Analytics') {
            steps {
                parallel Coverage: {
                    // Generate Code Coverage report
                    sh '/usr/local/bin/slather coverage --jenkins --html --scheme TimeTable TimeTable.xcodeproj/'

                    // Publish coverage results
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'html', reportFiles: 'index.html', reportName: 'Coverage Report'])


                }, Checkstyle: {

                    // Generate Checkstyle report
                    sh '/usr/local/bin/swiftlint lint --reporter checkstyle > checkstyle.xml || true'

                    // Publish checkstyle result
                    step([$class: 'CheckStylePublisher', canComputeNew: false, defaultEncoding: '', healthy: '', pattern: 'checkstyle.xml', unHealthy: ''])
                }, failFast: true
            }
        }

        //stage ('Notify') {
        //    steps {
        //        // Send slack notification
        //        slackSend channel: '#my-team', message: 'Time Table - Successfully', teamDomain: 'my-team', token: 'my-token'
        //    }
        //}
    }
}
