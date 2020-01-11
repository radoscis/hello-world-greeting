node('docker') {
    stage('Poll') {
        checkout scm
    }
    stage('Build & Unit test'){
        sh 'mvn clean verify -DskipITs=true';
        junit '**/target/surefire-reports/TEST-*.xml'
        archive 'target/*.jar'
    }
    stage('Static Code Analysis'){
        sh '''
            mvn clean verify sonar:sonar -Dsonar.projectName=jenkinsproject -Dsonar.projectKey=jenkinsproject -Dsonar.host.url=http://192.168.1.25:9000 -Dsonar.projectVersion=$BUILD_NUMBER
            '''
    }
    stage ('Integration Test'){
        sh 'mvn clean verify -Dsurefire.skip=true'
        junit '**/target/failsafe-reports/TEST-*.xml'
        archiveArtifacts 'target/*.jar', fingerprint: true
    }
    stage ('Publish'){
        def server = Artifactory.server 'artifactory-local'
        def uploadSpec = """{
        "files": [
        {
        "pattern": "target/hello-0.0.1.war",
        "target": "example-project/${BUILD_NUMBER}/",
        "props": "Integration-Tested=Yes;Performance-Tested=No"
        }
        ]
        }"""
        server.upload(uploadSpec)
    }
}