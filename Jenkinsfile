pipeline
{
    agent any

    stages
    {
        stage ('Build')
        {
        agent { docker { image 'maven:3-alpine' } }
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
