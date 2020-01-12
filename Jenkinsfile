node('docker') {
    stage('Poll') {
        checkout scm
    }
    stage('Build & Unit test'){
        sh 'mvn clean verify -DskipITs=true';
        junit '**/target/surefire-reports/TEST-*.xml'
        archiveArtifacts '**/target/*.war'
    }
    stage('Static Code Analysis'){
        sh '''
            mvn clean verify sonar:sonar \
            -Dsonar.projectName=jenkinsproject \
            -Dsonar.projectKey=jenkinsproject \
            -Dsonar.host.url=http://192.168.1.25:9000 \
            -Dsonar.projectVersion=$BUILD_NUMBER
            '''
    }
    stage ('Integration Test'){
        sh 'mvn clean verify -Dsurefire.skip=true'
        junit '**/target/failsafe-reports/TEST-*.xml'
        archiveArtifacts '**/target/*.war'
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
    stash includes: 'target/hello-0.0.1.war, src/pt/Hello_World_Test_Plan.jmx', name: 'binary'
}

node('docker_pt') {
    stage ('Start Tomcat'){
        sh '''cd /home/jenkins/tomcat/bin
        ./startup.sh''';
    }
    stage ('Deploy '){
        unstash 'binary'
        sh 'cp target/hello-0.0.1.war /home/jenkins/tomcat/webapps/';
    }
    stage ('Performance Testing'){
        sh '''cd /opt/jmeter/bin/
        ./jmeter.sh -n -t $WORKSPACE/src/pt/Hello_World_Test_Plan.jmx -l
        $WORKSPACE/test_report.jtl''';
        //step([$class: 'ArtifactArchiver', artifacts: '**/*.jtl'])
        archiveArtifacts '**/*.jtl'
    }
    stage ('Promote build in Artifactory'){
        withCredentials([usernameColonPassword(credentialsId:
        'artifactory-admin', variable: 'credentials')]) {
            sh '''
            curl -u${credentials} -X PUT
            "http://192.168.1.25:80/artifactory/api/storage/example-project/
            ${BUILD_NUMBER}/hello-0.0.1.war?properties=Performance-
            Tested=Yes"
            '''
        }
    }
}