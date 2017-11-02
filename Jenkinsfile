node {
    def server = Artifactory.server 'artifa' //newServer url: 'http://172.17.0.3:8081/artifactory', credentialsId: 'admp' //username: 'admin', password: 'password'
    def rtMaven = Artifactory.newMavenBuild()
    def buildInfo

    stage ('Git Clone') {
        git url: 'https://github.com/t3hlulzz/jenkins_wm.git'
    }

    stage ('Artifactory configuration') {
        rtMaven.tool = 'M3' // Tool name from Jenkins configuration
        rtMaven.deployer releaseRepo: 'libs-release-local', snapshotRepo:'libs-snapshot-local', server: server
        rtMaven.resolver releaseRepo: 'libs-release', snapshotRepo:'libs-snapshot', server: server
        buildInfo = Artifactory.newBuildInfo()
    }

    stage ('Maven goes to war') {
        docker.image('maven').inside {
            withEnv(['JAVA_HOME=/docker-java-home', 'MAVEN_HOME=/usr/share/maven']) { // Java/Maven home of the container
                rtMaven.run pom: 'pom.xml', goals: 'clean install', buildInfo: buildInfo
                buildInfo.env.capture = true
            }
        }
    }

    stage ('Publish build info') {
        server.publishBuildInfo buildInfo
    }

    stage ('Testing')
    {
      sh 'echo Testing 1..2...3'
    }

    // stage ('XRay scan')
    // {
    //   server.publishBuildInfo buildInfo
    //   def scanConfig = [
    //   'buildName'      : buildInfo.name,
    //   'buildNumber'    : buildInfo.number
    //   ]
    //   def scanResult = server.xrayScan scanConfig
    //   echo scanResult as String
    // }

    stage ('Promote build to release repo')
    {
    def promotionConfig = [
    // Mandatory parameters
    'buildName'          : buildInfo.name,
    'buildNumber'        : buildInfo.number,
    'targetRepo'         : 'libs-release-local',

    // Optional parameters
    'comment'            : 'Promotion',
    'sourceRepo'         : 'libs-snapshot-local',
    'status'             : 'Released',
    'includeDependencies': true,
    'copy'               : true
      ]
      server.promote promotionConfig
    }

    stage ('Download war from release')
    {
      def downloadSpec = """{
        "files": [
        {
          "pattern": "libs-release-local/com/*.war",
          "target": "war-local/",
          "flat": "true"
        }
          ]
          }"""
        server.download(downloadSpec)
    }

    stage ('Build docker image')
    {
      sh 'docker build . -t warapp:build-${BUILD_NUMBER}'
    }

}
