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

    stage ('Build with maven') {
        docker.image('maven').inside {
            withEnv(['JAVA_HOME=/docker-java-home', 'MAVEN_HOME=/usr/share/maven']) { // Java/Maven home of the container
                rtMaven.run pom: 'pom.xml', goals: 'clean install', buildInfo: buildInfo
                buildInfo.env.capture = true
            }
        }
    }

    // stage ('Testing with sonar')
    // {
    //   docker.image('sonarqube').inside
    //   {
    //     sh 'mvn sonar:sonar'
    //   }
    // }

    stage ('Publish build info') {
        server.publishBuildInfo buildInfo
    }

    stage ('Testing')
    {
      sh 'echo Testing 1..2...3'
    }

    stage ('Promote build to release repo')
    {
    def promotionConfig = [

    'buildName'          : buildInfo.name,
    'buildNumber'        : buildInfo.number,
    'targetRepo'         : 'libs-release-local',

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

    stage ('Tag and push docker image')
    {
      sh 'docker tag warapp:build-${BUILD_NUMBER} 172.17.0.4:5001/warapp:build-${BUILD_NUMBER}'
      sh 'docker push 172.17.0.4:5001/warapp:build-${BUILD_NUMBER}'
    }

    stage ('Promote docker image to release repo')
    {
      sh 'docker tag 172.17.0.4:5001/warapp:build-${BUILD_NUMBER} 172.17.0.4:5002/warapp:build-${BUILD_NUMBER}'
      sh 'docker push 172.17.0.4:5002/warapp:build-${BUILD_NUMBER}'
    }

    stage ('Cleanup images')
    {
      sh 'docker rmi warapp:build-${BUILD_NUMBER} -f'
      sh 'docker rmi 172.17.0.4:5001/warapp:build-${BUILD_NUMBER} -f'
      sh 'docker rmi 172.17.0.4:5002/warapp:build-${BUILD_NUMBER} -f'
    }

}
