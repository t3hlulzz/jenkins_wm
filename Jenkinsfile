pipeline
{
    agent { docker { image 'maven:3-alpine' } }

    stages
    {
        stage ('Build')
        {

            steps
            {
            sh 'mvn clean install -DbuildNumber=${BUILD_NUMBER}'
            }
        }
        stage ('Test')
        {
        agent any
            steps
            {
            sh 'echo Testing 1..2..3'
            }
        }
    }
}
