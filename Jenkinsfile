pipeline {
    agent any
	
    stages {
        stage('Init') {
            steps {
                deleteDir() // Comenzar con workspace limpio
                git branch: "develop",
                    credentialsId: "************************************",
                    url: "https://github.com/javiertuya/samples-test-spring.git"
            }
        }

        stage('test') {
            steps {
                echo "****** Build and test"
                // Para selenoid:
                // la url del driver es usa el nombre del contaienr porque comparte la red con este slave
                // la url de la aplicación debe ser la ip de este ejecutor
                // No configura browsers.json porque el servicio de selenoid está siempre disponible
                sh "echo 'remote.web.driver.url=http://selenoid:4444/wd/hub' > samples-test-spring.properties"
                sh "echo \"application.url=http://`(hostname -i)`\" >> samples-test-spring.properties"

                // Ejecuta maven, evitando que falle el build si fallan los tests,
                // luego los reports junit establecerán el estado inestable si procede
                sh "mvn clean verify -Dmaven.test.failure.ignore=true -U --no-transfer-progress"
            }
        }

        stage('report') {
            steps {
                echo "****** Publishing reports"
                // Resto del código de publicación de informes
            }
        }

        stage ('dependency-check') {
            steps {
                lock("dependency-check") {
                    echo "****** run OWASP dependency check"
                    def depCheckProg="/usr/share/dependency-check/bin/dependency-check.sh"
                    def depCheckParam="target/*-deploy.jar"
                    sh "${depCheckProg} --scan ${depCheckParam} --out ./target --format HTML --format JSON"
                    publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: false,
                        reportDir: "target", reportFiles: 'dependency-check-report.html',
                        reportName: 'Dependency Check'])
                    archiveArtifacts artifacts:'target/dependency-check-report.html', allowEmptyArchive:true
                }
            }
        }

        stage ('sonarqube') {
            steps {
                echo "****** SonarQube analysis"
                def scannerHome = tool 'SonarQube Scanner Linux';
                def sonarParams="-Dsonar.projectKey=my:samples-test-spring"
                withSonarQubeEnv('songis') {
                    sh "${scannerHome}/bin/sonar-scanner ${sonarParams}"
                }
            }
        }
    }
}
