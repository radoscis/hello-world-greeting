node('docker') {

stage('Pool') {
    scm checkout
}

stage('Build and Unit Test') {
    sh 'mvn clean verify -DskipITs=true';
    junit '**/target/surefire-reports/TEST-*.xml'
    archive 'target/*.jar'
}

stage('Static Code Analysis') {
    sh 'mvn clean verify sonar:sonar 
    -Dsonar.projectName=jenkinsproject 
    -Dsonar.projectKey=jenkinsproject 
    -Dsonar.projectVersion=$BUILD_NUMBER';
}

stage('Integration Tests') {
    sh 'mvn clean verify -Dsurefire.skip=true';
    junit '**/target/failsafe-reports/TEST-*.xml'
    archive 'target/*.jar'
}

stage ('Publish'){
    def server = Artifactory.server 'artifactory-local'
    def uploadSpec = """{
    "files": [
    {
    "pattern": "target/hello-0.0.1.war",
    "target": "helloworld-greeting-project/${BUILD_NUMBER}/",
    "props": "Integration-Tested=Yes;Performance-Tested=No"
    }
    ]
    }"""
    server.upload(uploadSpec)
}
}
