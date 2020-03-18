
node {
    def mvnHome

	stage('Preparation') {
        git url: 'https://github.com/GustyWinds24/WebApp.git', credentialsId: 'GitHubLogin'
        mvnHome = 'maven 3.3.9'
    }

	/*stage('Edit URL IPs') {
        // DB IP address change
        sh "sed -i -e 's/[0-9]\\{1,3\\}\\.[0-9]\\{1,3\\}\\.[0-9]\\{1,3\\}\\.[0-9]\\{1,3\\}/3.21.122.98/g' src/test/java/servlet/cancelpage.java"
        sh "sed -i -e 's/[0-9]\\{1,3\\}\\.[0-9]\\{1,3\\}\\.[0-9]\\{1,3\\}\\.[0-9]\\{1,3\\}/3.21.122.98/g' src/test/java/servlet/createpage.java"
        sh "sed -i -e 's/[0-9]\\{1,3\\}\\.[0-9]\\{1,3\\}\\.[0-9]\\{1,3\\}\\.[0-9]\\{1,3\\}/3.21.122.98/g' src/test/java/servlet/viewticket.java"
        // QA env IP address change
        sh "sed -i -e 's/[0-9]\\{1,3\\}\\.[0-9]\\{1,3\\}\\.[0-9]\\{1,3\\}\\.[0-9]\\{1,3\\}/3.21.122.98/g' functionaltest/src/test/java/functionaltest/ftat.java"
        // Prod env IP address change
        sh "sed -i -e 's/[0-9]\\{1,3\\}\\.[0-9]\\{1,3\\}\\.[0-9]\\{1,3\\}\\.[0-9]\\{1,3\\}/3.12.197.50/g' Acceptancetest/src/test/java/acceptancetest/acat.java"
    }
    
    stage('Push URL changes') {
        sh "git add src/test/java/servlet/cancelpage.java"
        sh "git add src/test/java/servlet/createpage.java"
        sh "git add src/test/java/servlet/viewticket.java"
        sh "git add functionaltest/src/test/java/functionaltest/ftat.java"
        sh "git add Acceptancetest/src/test/java/acceptancetest/acat.java"
        sh "git commit -m 'Pushing IP edited files'"
        sh 'git push https://GustyWinds24:DangerousToUse123@github.com/GustyWinds24/WebApp.git'
        //sh 'git pull'
    }*/

    stage('Clone sources') {
        git url: 'https://github.com/GustyWinds24/WebApp.git'
    }

    stage('Build triggers') {
        properties([[$class: 'JiraProjectProperty'], pipelineTriggers([pollSCM('H/2 * * * *')])])
    }

    stage('Build static-code-analysis job') { 
        withEnv( ["PATH+MAVEN=${tool mvnHome}/bin"] ) {
            withSonarQubeEnv(credentialsId: 'sonarqubetoken', installationName: 'SonarQube') {
                sh 'mvn clean "package" "sonar:sonar" -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.sources=. -Dsonar.tests=. -Dsonar.inclusions=**/test/java/servlet/createpage_junit.java -Dsonar.test.exclusions=**/test/java/servlet/createpage_junit.java -Dsonar.login=admin -Dsonar.password=admin'
           }
        }
    }

    def buildInfo

	stage('Build compile-web-app') {
        withEnv( ["PATH+MAVEN=${tool mvnHome}/bin"] ) {
            sh 'mvn -f pom.xml compile'
        }
    }

	stage('build deploy-to-qa') {
        deploy adapters: [tomcat7(credentialsId: 'tomcatid', path: '', url: 'http://3.21.122.98:8080/')], contextPath: '/QAWebapp', war: '**/*.war'
    }

	stage('build functional-testing') {
        withEnv( ["PATH+MAVEN=${tool mvnHome}/bin"] ) {
            sh 'mvn -f functionaltest/pom.xml test'
        }
    }

    stage('Publish functional testing HTML reports') {
        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\functionaltest\\target\\surefire-reports\\', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: ''])
    }

    // Get Artifactory server instance, defined in the Artifactory Plugin administration page.
    def server = Artifactory.server "tgdevops.jfrog.io"
    // Create an Artifactory Maven instance.
    def rtMaven = Artifactory.newMavenBuild()
    
    
    rtMaven.tool = "maven"

    /*stage('Clone sources') {
        git url: 'https://github.com/GustyWinds24/WebApp.git'
    }*/

    stage('Artifactory configuration') {
        // Tool name from Jenkins configuration
        rtMaven.tool = "maven 3.3.9"
        // Set Artifactory repositories for dependencies resolution and artifacts deployment.
        rtMaven.deployer releaseRepo:'libs-release-local', snapshotRepo:'libs-snapshot-local', server: server
        rtMaven.resolver releaseRepo:'libs-release', snapshotRepo:'libs-snapshot', server: server
    }

    stage('Maven build') {
        buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean install'
    }

    stage('Publish build info') {
        server.publishBuildInfo buildInfo
    }

	// Here is where blazemeter is invoked
	/*stage('Build performance-testing') {
        blazeMeterTest credentialsId: 'blazemeterkey', testId: '7814049.taurus', workspaceId: '454266'
    }*/
	
    stage('Build production-deployment') {
        deploy adapters: [tomcat7(credentialsId: 'tomcatid', path: '', url: 'http://3.12.197.50:8080/')], contextPath: '/ProdWebapp', war: '**/*.war'
    }

    stage('Build sanity-test-production') {
        slackSend channel: '#devops', message: 'sanity-test-production - Started by upstream project "production-deployment"'
        withEnv( ["PATH+MAVEN=${tool mvnHome}/bin"] ) {
            sh 'mvn -f Acceptancetest/pom.xml test'
        }
        slackSend channel: '#devops', message: 'sanity-test-production - Success'
    }

	stage('Publish sanity test HTML reports') {
        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\Acceptancetest\\target\\surefire-reports\\', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: ''])
    }

    stage('Clone sources') {
        git url: 'https://github.com/GustyWinds24/docker-demo.git'
    }

	environment {
        repo = "tcsilptg/nodejswebapp_team2_test1"
        registryCredential = 'tcsilptgid'
        dockerImage = ''
    }

    stage('Build nodejs-app') {
        nodejs('nodejs') {
            sh 'npm install'
        }        
    }

    stage('Build image') {
        dockerImage = docker.build repo + ":$BUILD_NUMBER"
    }

    stage('Deploy image') {
        docker.withRegistry( '', registryCredential) {
            dockerImage.push()
        }
    }


    }
	 
