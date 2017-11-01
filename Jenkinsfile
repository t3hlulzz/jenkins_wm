
node {
    def server = Artifactory.newServer url: 'http://172.18.0.3:8081/artifactory', username: 'admin', password: 'password' //credentialsId: CREDENTIALS
    def rtMaven = Artifactory.newMavenBuild()
    def buildInfo

    stage ('Clone') {
        git url: 'https://github.com/jfrogdev/project-examples.git'
    }

    stage ('Artifactory configuration') {
        rtMaven.tool = 'M2' // Tool name from Jenkins configuration
        rtMaven.deployer releaseRepo: 'libs-release-local', snapshotRepo:'libs-snapshot-local', server: server
        rtMaven.resolver releaseRepo: 'libs-release', snapshotRepo:'libs-snapshot', server: server
        buildInfo = Artifactory.newBuildInfo()
    }

    stage ('Exec Maven') {
        docker.image('maven').inside {
            withEnv(['JAVA_HOME=/docker-java-home', 'MAVEN_HOME=/usr/share/maven']) { // Java/Maven home of the container
                rtMaven.run pom: 'maven-example/pom.xml', goals: 'clean install', buildInfo: buildInfo
            }
        }
    }

    stage ('Publish build info') {
        server.publishBuildInfo buildInfo
    }
}
