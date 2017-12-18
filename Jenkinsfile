/****************************** Environment variables ******************************/ 
def JobName	= null						// variable to get jobname 
def Sonar_project_name = null 							// varibale passed as SonarQube parameter while building the application
def robot_result_folder = null 				// variable used to store Robot Framework test results
def server = Artifactory.server 'server1'	// Artifactory server instance declaration. 'server1' is the Server ID given to Artifactory server in Jenkins
def buildInfo = null 								// variable to store build info which is used by Artifactory
def rtMaven = Artifactory.newMavenBuild()	// creating an Artifactory Maven Build instance
def Reason = "JOB FAILED"					// variable to display the build failure reason
def lock_resource_name = null 					// variable for storing lock resource name

// Reading jar file name from pom.xml //
def getMavenBuildArtifactName() {
 pom = readMavenPom file: 'pom.xml'
 return "${pom.artifactId}-${pom.version}.${pom.packaging}"
}

// Email Notifications template when Build succeeds //
def notifySuccessful(){
emailext (
 attachLog: true, attachmentsPattern: '*.html, output.xml', body: '''<span style=\'line-height: 22px; font-family: Candara; padding: 10.5px; font-size: 15px; word-break: break-all; word-wrap: break-word; \'>
 <h1><FONT COLOR=Green>$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS</FONT></h1><h2 style=\'color:#e46c0a\'>GitHub Details</h2>
 <B>${BUILD_LOG_REGEX, regex="Started by ", linesBefore=0, linesAfter=1, maxMatches=1, showTruncatedLines=false, escapeHtml=true}<br>
 ${BUILD_LOG_REGEX, regex="Checking out Revision", linesBefore=0, linesAfter=1, maxMatches=1, showTruncatedLines=false, escapeHtml=true}</B>
 <p><!-- ${SCRIPT, template="unit_test_results.groovy"} --></p>
 <p><br><br>${SCRIPT, template="sonarqube_template.groovy"}<br></p>
 <p><br><br><br><br><br><br><br><h2 style=\'color:#e46c0a; font-family: Candara;\'>Artifactory Details</h2>
 <b style=\'font-family: Candara;\'>${BUILD_LOG_REGEX, regex="http://padlcicdggk4.sw.fortna.net:8088/artifactory/webapp/*", linesBefore=0, linesAfter=0, maxMatches=1, showTruncatedLines=false, escapeHtml=true}<b></p>
 <p><br><br>${SCRIPT, template="robotframework_template.groovy"}</p>
 <p><br><br><br><br><br><br><br><h2><a href="$BUILD_URL">Click Here</a> to view build result</h2><br><h3>Please find below, the build logs and other files.</h3></p>
 </span>''', subject: '$DEFAULT_SUBJECT', to: 'yerriswamy.konanki@ggktech.com, sunil.boga@ggktech.com'
 )
}
 
// Email Notifications template when Build fails //
def notifyFailure(def Reason){
println "Failed Reason: ${Reason}"
emailext (
	attachLog: true, attachmentsPattern: '*.html, output.xml', body: '''<span style=\'line-height: 22px; font-family: Candara; padding: 10.5px; font-size: 15px; word-break: break-all; word-wrap: break-word; \'>
	<h1><FONT COLOR=red>$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS</FONT></h1>
  	<h1>${BUILD_LOG_REGEX, regex="Failed Reason:", linesBefore=0, linesAfter=0, maxMatches=1, showTruncatedLines=false, escapeHtml=true}</h1>
	<p><h2><a href="$BUILD_URL">Click Here</a> to view build result</h2><br><h3>Please find below, the build logs and other files.</h3></p>
	</span>''', subject: '$DEFAULT_SUBJECT', to: 'sneha.kailasa@ggktech.com'
	)
}

/****************************** Jenkinsfile execution starts here ******************************/
node {
	def content = readFile './.env'				// variable to store .env file contents
	Properties docker_properties = new Properties()	// creating an object for Properties class
	InputStream contents = new ByteArrayInputStream(content.getBytes());	// storing the contents
	docker_properties.load(contents)	
	contents = null
	try {
/****************************** Git Checkout Stage ******************************/
		stage ('Checkout') {
			Reason = "GIT Checkout Failed"
			checkout scm				
		}

// assigning the jarname to this variable aquired from pom.xml by below function //
		def jar_name = getMavenBuildArtifactName()

/****************************** Stage that creates lock variable and SonarQube variable ******************************/
		stage ('Reading Branch Varibles ')	{
			sh 'env >Jenkins_env'
			//sh 'ls'
			//sh'reading'
	        content = readFile './Jenkins_env'				// variable to store .env file contents
	        Properties jenkins_properties = new Properties()	// creating an object for Properties class
	        contents = new ByteArrayInputStream(content.getBytes());	// storing the contents
	        jenkins_properties.load(contents)	
	        contents = null
	        //sh'echo completed'
            Reason = "lockVar stage Failed"
            JobName = jenkins_properties.JOB_NAME
			//JobName = "testinglock2/latest"
            //Sonar_project_name = "testinglock2_latest"
           // lockVar = "testinglock2_latest"
            def git_branch = jenkins_properties.BRANCH_NAME
		//	sh 'echo startedreadingreading'
           	
			//sh 'echo reading failed'
			
			if(git_branch.startsWith('PR-'))	//if(JobName.contains('PR-'))
			{
                def target_branch = jenkins_properties.CHANGE_TARGET
				def index = JobName.indexOf("/");
				lock_resource_name = JobName.substring(0 , index)+"_"+"${target_branch}"
                Sonar_project_name = lock_resource_name + "PR"
				 //println index; println lock_resource_name; println Sonar_project_name;
			}
			else
			{
				 def index = JobName.indexOf("/");
				 lock_resource_name = JobName.substring(0 , index)+"_"+"${git_branch}"
				 Sonar_project_name = lock_resource_name
				// println index; println lock_resource_name; println Sonar_project_name;
			} 
		}
	
/****************************** Building the Application and performing SonarQube analysis ******************************/	
		stage ('Maven Build') {
			Reason = "Maven Build Failed"
			rtMaven.deployer server: server, snapshotRepo: 'fortna_snapshot', releaseRepo: 'fortna_release'			//Deploying artifacts to this repo //
			rtMaven.deployer.deployArtifacts = false																//this will not publish artifacts soon after build succeeds	//
			rtMaven.tool = 'maven'																					//Defining maven tool //
			// Maven build starts here //
			//withSonarQubeEnv {
				def mvn_version = tool 'maven'
				//echo "${mvn_version}"
				withEnv( ["PATH+MAVEN=${mvn_version}/bin"] ) {
					buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean install -Dmaven.test.skip=true' //$SONAR_MAVEN_GOAL -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.projectKey="$Sonar_project_name" -Dsonar.projectName="$Sonar_project_name"'
				}
			//}
		}

/****************************** Docker Compose and Robot Framework testing on container ******************************/
		stage ('Docker Deploy and RFW') {
			Reason = "Docker Deployment or Robot Framework Test cases Failed"
			lock(lock_resource_name) {
				// Docker Compose starts // 
				sh "jarfile_name=${jar_name} /usr/local/bin/docker-compose up -d"
				sh "sudo chmod 777 wait_for_robot.sh "
                sh './wait_for_robot.sh'
				robot_result_folder = docker_properties.robot_result_folder
				//sh 'echo /home/robot/${robot_result_folder}/report.html'
				step([$class: 'RobotPublisher',
					outputPath: "/home/robot/${robot_result_folder}",
					passThreshold: 0,
					unstableThreshold: 0,
					otherFiles: ""])
				// If Robot Framework test case fails, then the build will be failed //	
            //    println currentBuild.result
				if("${currentBuild.result}" == "FAILURE")
					 {	
						 sh ''' ./clean_up.sh
                         echo "after cleanup"
						 exit 1'''
					 } 
				// If it is a GitHub PR job, then this part doesn't execute //					 
				if(!(JobName.contains('PR-')))
				{
					 // ***** Stage for Deploying artifacts to Artifactory ***** //				
			/*	stage ('Artifacts Deployment'){		
						Reason = "Artifacts Deployment Failed"
						rtMaven.deployer.deployArtifacts buildInfo
						server.publishBuildInfo buildInfo
					}	*/
					// ***** Stage for Publishing Docker images ***** //							
					stage ('Publish Docker Images'){
						Reason = "Publish Docker Images Failed"
						def cp_index = docker_properties.cp_image_name.indexOf(":");								
						def cpImageName = docker_properties.cp_image_name.substring(0 , cp_index)+":latest"
						def om_index = docker_properties.om_image_name.indexOf(":");
						def omImageName = docker_properties.om_image_name.substring(0 , om_index)+":latest"
						sh """
							docker tag ${docker_properties.om_image_name} swamykonanki/${docker_properties.om_image_name}
							docker tag ${docker_properties.om_image_name} swamykonanki/${omImageName}
							docker tag ${docker_properties.cp_image_name} swamykonanki/${docker_properties.cp_image_name}
							docker tag ${docker_properties.cp_image_name} swamykonanki/${cpImageName}
							"""
						/*	docker.withRegistry("https://index.docker.io/v1/", 'DockerCredentialsID'){
								def customImage1 = docker.image("swamykonanki/${docker_properties.om_image_name}")
								customImage1.push()
								def customImage2 = docker.image("swamykonanki/${omImageName}")
								customImage2.push()
								def customImage3 = docker.image("swamykonanki/${docker_properties.cp_image_name}")
								customImage3.push()
								def customImage4 = docker.image("swamykonanki/${cpImageName}")
								customImage4.push()
							}
							sh """docker logout""" 
					*/
					}  //docker push
				
					// ***** Stage for triggering CD pipeline ***** //				
					stage ('Starting QA job') {
					Reason = "Trriggering downStream Job Failed"
                    Job_name = Sonar_project_name + "_QA"
		   			 	build job: Job_name//, parameters: [[$class: 'StringParameterValue', name: 'var1', value: 'var1_value']]
					} 
				}     //if loop
				sh './clean_up.sh'	
			}	                   //lock			
		}							// Docker Deployment and RFW stage ends here //

/****************************** Stage for artifacts promotion ******************************/
	/*	stage ('Build Promotions') {
			Reason = "Build Promotions Failed"
			def promotionConfig = [
				// Mandatory parameters
				'buildName'          : buildInfo.name,
				'buildNumber'        : buildInfo.number,
				'targetRepo'         : 'fortna_release',
	 
				// Optional parameters
				'comment'            : 'PROMOTION SUCCESSFULLY COMPLETED',
				'sourceRepo'         : 'fortna_snapshot',
				'status'             : 'Released',
				'includeDependencies': false,
				'copy'               : false,
				'failFast'           : true
			]
	 
			// Interactive promotion of Builds in Artifactory server from Jenkins UI //
			Artifactory.addInteractivePromotion server: server, promotionConfig: promotionConfig, displayName: "Promotions Time" //this need human interaction to promote
		}
	
/****************************** Stage for creating reports for SonarQube Analysis ******************************
		stage ('Reports creation') {
			Reason = "Reports creation Failed"
			sh '''sleep 15s
			curl "http://10.240.17.12:9000/sonar/api/resources?resource=$JOB_NAME&metrics=bugs,vulnerabilities,code_smells,duplicated_blocks" > output.json
			sleep 10s'''
		}*/

/****************************** Stage for sending Email Notifications when Build succeeds ******************************/	
		stage ('Email Notifications') {
			notifySuccessful() 
		}
	}
	
catch(Exception e)
	{
		currentBuild.result = "FAILURE"
		notifyFailure(Reason)
		sh 'exit 1'
	}
}
