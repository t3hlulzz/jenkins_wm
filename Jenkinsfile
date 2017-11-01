
node {
    def server = Artifactory.newServer url: 'http://172.18.0.3:8081/artifactory', credentialsId: admp //username: 'admin', password: 'password'
    def rtMaven = Artifactory.newMavenBuild()
    def buildInfo

    stage ('Clone') {
        git url: 'https://github.com/t3hlulzz/jenkins_wm.git' //'https://github.com/jfrogdev/project-examples.git'
    }

    stage ('Artifactory configuration') {
        rtMaven.tool = 'M2' // Tool name from Jenkins configuration
        rtMaven.deployer releaseRepo: 'libs-release-local', snapshotRepo:'libs-snapshot-local', server: server
        rtMaven.resolver releaseRepo: 'libs-release', snapshotRepo:'libs-snapshot', server: server
        buildInfo = Artifactory.newBuildInfo()
    }

    stage ('Exec Maven') {
        docker.image('maven').inside {
            withEnv(['JAVA_HOME=/docker-java-home']) { // Java/Maven home of the container
                rtMaven.run pom: 'pom.xml', goals: 'clean install', buildInfo: buildInfo
            }
        }
    }

    stage ('Publish build info') {
        server.publishBuildInfo buildInfo
    }
}
