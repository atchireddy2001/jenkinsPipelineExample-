def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]

pipeline
{
    agent any
    tools
    {
        maven 'MAVEN3'
        jdk 'OracleJDK11'
    }
    
    //environment{

    //}

    stages
    {
        stage('Fetching code from GIT')
        {
            steps
            {
                git branch: 'main', url: 'https://github.com/hkhcoder/vprofile-project.git'
            }
        }

        stage('Building app & Post step Archiving war file of build')
        {
            steps
            {
                sh 'mvn install -DskipTests'
            }

            post
            {
                success
                {
                     archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage('JUnit Testing')
        {
            steps
            {
                sh 'mvn test'
            }
        }

        stage('CODE_ANALYSIS -- using checkstyle')
        {
            steps
            {
                sh 'mvn checkstyle:checkstyle'
            }
            post
            {
                success
                {
                    echo 'Generated checkstyle results now you can check for directoiry in workshop'
                }
            }
        }

        stage('CODE_ANALYSIS -- using SonarScanner & uploading junit jacaco checkstyle results into sonar')
        {
            environment
            {
                scannerHome = tool 'sonar4.7'   //sonarQube scanner tool name configured in tools section
            }
            steps 
            {
                withSonarQubeEnv('sonarServer') 
                {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }

        stage('Quality gate - config in sonar server')
        {
            steps
            {
                timeout(time: 10, unit: 'MINUTES') 
                {
                     waitForQualityGate abortPipeline: true
                }
            }
            
        }

        stage('Uploading Artifact in Nexus repo')
        {
            steps
            {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: '172.31.93.138:8081',
                    groupId: 'Sample',
                    version: "${env.BUILD_ID}_${env.BUILD_TIMESTAMP}",
                    repository: 'vprofile-repo',
                    credentialsId: 'credfornexus',
                    artifacts: [
                        [artifactId: 'vprofileApp', 
                        file: 'target/vprofile-v2.war',
                        type: 'war']
                    ]
                )
            }
        }

    }

    post
    {
        always{
            echo 'SLACK Notification'
            slackSend channel: '#cicdjenkinsnotification',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }

}
