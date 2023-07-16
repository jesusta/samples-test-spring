pipeline {
    agent any

    stages {
        stage('Init') {
            steps {
                deleteDir()
                checkout([$class: 'GitSCM',
                          branches: [[name: 'develop']],
                          userRemoteConfigs: [[url: 'https://github.com/jesusta/samples-test-spring.git']],
                          extensions: [[$class: 'LocalBranch', localBranch: 'develop']]])
            }
        }

        stage('Build and Test') {
            steps {
                echo "****** Build and test"
                sh "echo 'remote.web.driver.url=http://selenoid:4444/wd/hub' > samples-test-spring.properties"
                sh "echo \"application.url=http://$(hostname -i)\" >> samples-test-spring.properties"
                sh "mvn clean verify -Dmaven.test.failure.ignore=true -U --no-transfer-progress"
            }
        }

        stage('Publish Reports') {
            steps {
                echo "****** Publishing reports"
                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    junit '**/target/surefire-reports/TEST-*.xml,**/target/failsafe-reports/TEST-*.xml'
                }

                script {
                    def reportDir = 'target/site'

                    publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: false,
                                 reportDir: reportDir, reportFiles: 'surefire-report.html',
                                 reportName: 'Surefire Report'])
                    publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: false,
                                 reportDir: reportDir, reportFiles: 'failsafe-report.html',
                                 reportName: 'Failsafe Report'])
                    publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: false,
                                 reportDir: reportDir + '/jbehave/view', reportFiles: 'reports.html',
                                 reportName: 'JBehave Report'])
                    publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: false,
                                 reportDir: reportDir + '/jacoco', reportFiles: 'index.html',
                                 reportName: 'JaCoCo Report'])
                    publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: false,
                                 reportDir: reportDir + '/junit-frames', reportFiles: 'index.html',
                                 reportName: 'JUnit Report (frames)'])
                    publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: false,
                                 reportDir: reportDir + '/junit-noframes', reportFiles: 'junit-noframes.html',
                                 reportName: 'JUnit Report (noframes)'])
                }

                archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
                archiveArtifacts artifacts: '**/target/*.log', allowEmptyArchive: true
                archiveArtifacts artifacts: '**/target/*.html', allowEmptyArchive: true
                archiveArtifacts artifacts: '**/target/site/screenshot/*.png', allowEmptyArchive: true
            }
        }

        stage('Dependency Check') {
            steps {
                lock('dependency-check') {
                    echo "****** Run OWASP dependency check"
                    def depCheckProg = '/usr/share/dependency-check/bin/dependency-check.sh'
                    def depCheckParam = 'target/*-deploy.jar'
                    sh "${depCheckProg} --scan ${depCheckParam} --out ./target --format HTML --format JSON"
                    publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: false,
                                 reportDir: 'target', reportFiles: 'dependency-check-report.html',
                                 reportName: 'Dependency Check'])
                    archiveArtifacts artifacts: 'target/dependency-check-report.html', allowEmptyArchive: true
                }
            }
        }

       
    }
}
