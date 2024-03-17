# jenkins-sonarqube
1.	Install Docker Desktop:
•	Download and install Docker Desktop for Windows from the official Docker website.
•	Follow the installation instructions provided by Docker.
2.	Run SonarQube Container:
•	Open a terminal or command prompt.
•	Pull the SonarQube Docker image: docker pull sonarqube.
•	Run the SonarQube container:
cssCopy code
docker run -d --name sonarqube -p 9000:9000 sonarqube 
•	Access SonarQube web UI at http://localhost:9000 in your browser.
•	Follow the setup wizard to configure SonarQube.
3.	Set Up Jenkins:
•	Download and install Jenkins for Windows from the official Jenkins website.
•	Follow the installation instructions provided by Jenkins.
4.	Configure Jenkins:
•	Open Jenkins in your browser (http://localhost:8080 by default).
•	Follow the initial setup wizard to set up Jenkins.
•	Install necessary plugins:
•	Git Plugin: For integrating with Git repositories.
•	Pipeline Maven Integration : to configure pipeline job by calling sh mvn or bat mvn.
•	SonarQube Scanner Plugin: For integrating SonarQube analysis into Jenkins pipelines.
•	Generic Webhook Trigger Plugin:
1.	 The Generic Webhook Trigger Plugin enables Jenkins to receive webhook notifications from external systems, such as SonarQube.
2.	It allows Jenkins to trigger pipeline jobs automatically in response to webhook events received from SonarQube.
3.	With this plugin, Jenkins can listen for webhook notifications sent by SonarQube upon the completion of analysis tasks, such as analysis success or failure.
•	Configure Jenkins Global Tool Configuration to set up Maven and SonarQube Scanner.
•	Tools =>Configure tools ,thier location and automatic installer =>Maven installations => name (maven) and MAVEN_HOME
•	manage jenkins=> System=>Configure global setting and paths => SonarQube servers => SonarQube installations => Name (sonar) and server url (http://localhost:9002)
5.	Create Jenkins Pipeline:
•	Open Jenkins dashboard.
•	Create a new pipeline project.
•	Configure the pipeline script similar to the one you provided, ensuring the correct SonarQube URL and Git repository URL are used.
```

    pipeline {
        agent any
        stages {
            stage('SCM') {
                steps {
                     // this step will checkouut project to jenkins local workspace.
                    git url: 'https://github.com/somajoshi88/star-agile-insurance-project.git'
                }
            }
            stage('build && SonarQube analysis') {
                steps {

                    // name 'Sonar' is configured => manage jenkins=> System=>Configure global setting and paths => 
                    // SonarQube servers => SonarQube installations => Name (sonar) and server url (http://localhost:9002) and sonarqube token(generate from sonarqube and configure in jenkins credentials and use var here)
                    // manage jenkins=> Configure tools =>SonarQube Scanner installations=>Add SonarQube Scanner=>Install automatically =>Install from Maven Central

                    withSonarQubeEnv('Sonar') {
                    
                        // name 'maven' is configured =>Tools =>Configure tools ,thier location and automatic installer =>Maven installations => name (maven) and MAVEN_HOME  
                        withMaven(maven:'maven') {

                            // following step will build jenkins local workspace project and request sonar to review target folder.
                            bat 'mvn clean package sonar:sonar -Dsonar.host.url=http://localhost:9002 -Dsonar.java.binaries=target'

                        }
                    }
                }
            }
            stage("Quality Gate") {
                steps {
                    timeout(time: 1, unit: 'HOURS') {
                        // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                        // true = set pipeline to UNSTABLE, false = don't
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }
    }

```
6.	Set Up Webhook in SonarQube:
•	Access SonarQube web UI (http://localhost:9000).
•	Navigate to project > project setting > Webhooks.
•	Add a new webhook with the URL of your Jenkins server followed by /sonarqube-webhook/.
•	create => name ,URL =>http://192.168.1.14:8080/sonarqube-webhook ( need to put local machine IP as it does not accept loopback localhost(127.0.0.1) . ,no need to put secret for dev env.
7.	Run Jenkins Pipeline:
•	Trigger the Jenkins pipeline manually or configure it to run automatically on code commits.
•	Jenkins will clone the Git repository, run the build, perform SonarQube analysis, and wait for the Quality Gate status.
8.	Verify Integration:
•	Check Jenkins pipeline execution logs to ensure that the build, SonarQube analysis, and Quality Gate check are successful.
•	Verify in SonarQube that the project analysis results are displayed correctly.
9.	Troubleshooting:
•	If the Jenkins job gets stuck, ensure that Jenkins can access SonarQube and vice versa.
•	Check network configurations and firewall settings to allow communication between Jenkins and SonarQube.
•	Verify that the webhook URL in SonarQube is correct and points to the Jenkins server.
By following these steps and installing the required plugins, you should be able to set up SonarQube with Docker, Jenkins, and integrate them to review a Git repository using a Jenkins pipeline. Adjust the configurations as needed based on your specific environment and requirements.

