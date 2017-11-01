node {
    def server = Artifactory.server 'artifa' //newServer url: 'http://172.17.0.3:8081/artifactory', credentialsId: 'admp' //username: 'admin', password: 'password'
    def rtMaven = Artifactory.newMavenBuild()
    def buildInfo

    stage ('Clone') {
        git url: 'https://github.com/t3hlulzz/jenkins_wm.git'
    }

    stage ('Artifactory configuration') {
        rtMaven.tool = 'M3' // Tool name from Jenkins configuration
        rtMaven.deployer releaseRepo: 'libs-release-local', snapshotRepo:'libs-snapshot-local', server: server
        rtMaven.resolver releaseRepo: 'libs-release', snapshotRepo:'libs-snapshot', server: server
        buildInfo = Artifactory.newBuildInfo()
    }

    stage ('Exec Maven') {
        docker.image('maven').inside {
            withEnv(['JAVA_HOME=/docker-java-home', 'MAVEN_HOME=/usr/share/maven']) { // Java/Maven home of the container
                rtMaven.run pom: 'pom.xml', goals: 'clean install', buildInfo: buildInfo
                buildInfo.env.capture = true
            }
        }
    }

    stage ('Stash')
    {
      stash includes: '**/target/*.war', name: 'app'
    }

    stage ('Publish build info') {
        server.publishBuildInfo buildInfo
    }
}
