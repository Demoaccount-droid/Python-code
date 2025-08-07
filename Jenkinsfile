def finalreportURL
def selectedDevice 
def lockableResource(){
                      def JENKINS_USER = 'admin'
                      def JENKINS_PASS = 'admin'
    
                        def authString = "${JENKINS_USER}:${JENKINS_PASS}".bytes.encodeBase64().toString()
                        def jobURL = 'http://107.108.213.7:8080/lockable-resources/api/json'
                        def connection = new URL(jobURL).openConnection() 
    	 	            connection.setRequestProperty("Authorization","Basic ${authString}")
    	 	            connection.setRequestProperty("Accept","applicatio/json")
                		def inputStream = connection.inputStream
                		def response = inputStream.text
                		def json = new groovy.json.JsonSlurper().parseText(response)
    		            
    		            def ubuntu17FreeResources = json.resources.findAll { it.free && !it.reserved && it.description == "ubuntu-17" && it.labels == "8865" }.collect{it.name}
    		            //def ubuntu2204freeResources = json.resources.findAll { it.free || !it.free}.collect{it.name}
    		            echo "Free Resources in Nodes: ${ubuntu17FreeResources}"
    		            def shuffled = ubuntu17FreeResources.toList()
    		            Collections.shuffle(shuffled)
    		            def randomItem = shuffled[0]
    		            echo "This resource can be used for the build: ${randomItem}"
    		            return randomItem
}
// selectedDevice = lockableResource()
// def getDevice(def selectedDevice){
//     return selectedDevice
// }

pipeline {
    agent {
        node {
            label 'built-in'
        }
    }
    environment {
        
        // Workspace Environment
        
        SUMD_MAIN_WORKSPACE = "$WORKSPACE/sumd_main"
        SUMD_MX2024_WORKSPACE = "$WORKSPACE/sumd_mx2024"
        SUMD_MX2025_WORKSPACE = "$WORKSPACE/sumd_mx2025"
		
		SUMD_SOCL_MAIN_WORKSPACE = "$WORKSPACE/sumd_socl_main"
        SUMD_SOCL_MX2024_WORKSPACE = "$WORKSPACE/sumd_socl_mx2024"
        SUMD_SOCL_MX2025_WORKSPACE = "$WORKSPACE/sumd_socl_mx2025"
        
        ANGLE_MAIN_WORKSPACE = "$WORKSPACE/angle_main"
        ANGLE_RELEASE_WORKSPACE = "$WORKSPACE/angle_release"
        ANGLE_MX2025_WORKSPACE = "$WORKSPACE/angle_mx2025"
        
		ONE_WORKSPACE="$WORKSPACE/one_ring"
        
		// REFERENCE REPOSITORY
        
        // sumd
		SUMD_MAIN_REFERENCE = "/var/lib/jenkins/workspace/sumd_main_trishul_mirror/sumd_main"
        SUMD_MX2024_REFERENCE = "/var/lib/jenkins/workspace/sumd_mx2024_trishul_mirror/sumd_mx2024"
        SUMD_MX2025_REFERENCE = "/var/lib/jenkins/workspace/sumd_mx2025_trishul_mirror/sumd_mx2025"
        
        //angle
        ANGLE_MAIN_REFERENCE="/var/lib/jenkins/workspace/angle_main_trishul_mirror/angle_main"
        ANGLE_RELEASE_REFERENCE="/var/lib/jenkins/workspace/angle_release_trishul_mirror/angle_release"
        ANGLE_MX2025_REFERENCE="/var/lib/jenkins/workspace/angle_mx2025_trishul_mirror/angle_mx2025"
        		
        // SARC-C Environment
        
        JAVA_TOOL_OPTIONS = "-Djavax.net.ssl.trustStore=/sarc-c/gpusw/env/cacerts -Djavax.net.ssl.trustStorePassword=changeit"
        SSL_CERT_FILE = "/sarc-c/gpusw/env/SARC-CA-SARC-Enterprise-CA-FullChain.pem"
        REQUESTS_CA_BUNDLE = "/sarc-c/gpusw/env/SARC-CA-SARC-Enterprise-CA-FullChain.pem"
        PATH = "$PATH:/opt/poetry/bin:/sarc-c/gpusw/SWV/bin/:/usr/local/android-sdk/platform-tools"
		
		CTS_SCRIPT_PATH="/var/lib/jenkins/scripts/CTS/one_ring/openGL_cts"
		
		// Packages path
		PACKAGES_PATH="/var/lib/jenkins/packages"
		BSP_PACKAGES_PATH="/var/lib/jenkins/userContent/bsp"
		BSP_SCRIPT_PATH="/var/lib/jenkins/scripts/bsp/fetch_bsp_script"
		
		// BSP Flashing Script Path
		DEVICE_FLASH_SCRIPT_PATH="/var/lib/jenkins/scripts/device_test/ssir_flash_script"
		RECOVER_SCRIPT_PATH="/var/lib/jenkins/scripts/device_test/recover_device"
		
		// Android devices
        ANDROID_SERIAL="$device"
        device = "$device"
        selectedDevice = lockableResource()
   
        USERCONTENT_DIR="/var/lib/jenkins/userContent/cts/openGL_cts_custom"
        REPORT_DIR="/var/lib/jenkins/workspace/openGL_cts_custom/one_ring/_ci_out"
    }
    parameters {
        choice(name:'sumd_branch', choices: ['mx/2024','mx/2025', 'main'], description: 'Enter the sumd branch to build the SW')
        
		choice(name:'angle_branch', choices: ['release','main','mx/2025'], description: 'Enter the angle branch to build the SW')
        
        string(name: 'sumd_patch', defaultValue: '', description: 'Enter the active refs/changes/<value> number (leave blank for TOT)')
        
		string(name: 'angle_patch', defaultValue: '', description: 'Enter the active refs/changes/<value> number (leave blank for TOT)')

		choice(name: 'test_type', choices: ['all', 'glcts:egl', 'glcts:gles3', 'glcts:gles2-khr', 'glcts:gles2', 'glcts:gles3-khr', 'glcts:gles31-khr', 'glcts:gles31', 'glcts:gles32-khr'], description: 'Select the Required test type for Testing')
		
		string(name: 'BSP_PACKAGE', defaultValue: '', description: 'Update the BSP for one ring repo')
		
		choice(name: 'android_version', choices: ["android16", "android14", "android15", "android17"], description: 'Select the Android type for Syncing')
        
		activeChoice choiceType: 'PT_SINGLE_SELECT', filterLength: 1, filterable: false, name: 'target_version', randomName: 'choice-parameter-981418929352697', script: groovyScript(fallbackScript: [classpath: [], oldScript: '', sandbox: true, script: 'return ["Something wrong with the option"]'], script: [classpath: [], oldScript: '', sandbox: true, script: 'return ["8855", "8845", "8865", "8875", "9945", "9955", "9965", "9975"]'])
		
		reactiveChoice choiceType: 'PT_SINGLE_SELECT', filterLength: 1, filterable: false, name: 'image_type', randomName: 'choice-parameter-2815002183367035', referencedParameters: 'target_version,android_version', script: groovyScript(fallbackScript: [classpath: [], oldScript: '', sandbox: true, script: ''], script: [classpath: [], oldScript: '', sandbox: true, script: '''	if (target_version=="8855" && android_version=="android15") {
           return [\'_essi_b_64_userdebug\',\'_v_userdebug\', \'_v_user\',\'_v\', \'_mx_delivery_v_eng\', \'_hwasan_v_userdebug\']
        }
			if (target_version=="8855" && android_version=="android16") {
				   return [\'_essi_b_64_userdebug\',\'_b_userdebug \']
				}
			if (target_version=="8865"  && android_version=="android16") {
				   return [\'_b\',\'_b_userdebug\',\'_b_user\',\'_mx_delivery_b_eng\']
				}
			if (target_version=="8845" &&  android_version=="android16"){
					return [\'_essi_b_64\',\'_essi_b_64_userdebug\']
				}
			if (target_version=="8845" && android_version=="android16") {
				   return [\'_essi_b_64\',\'_essi_b_64_userdebug\']
				}'''])
				
		choice(name: 'automaticDeviceSelection', choices: ['Yes', 'No'], description: 'Select if you want to automatically select a device')
        
        reactiveChoice choiceType: 'PT_SINGLE_SELECT', filterLength: 1, filterable: false, name: 'device', randomName: 'choice-parameter-981418934889274', referencedParameters: 'target_version,automaticDeviceSelection', script: groovyScript(fallbackScript: [classpath: [], oldScript: '', script: ''], script: [classpath: [], oldScript: '', 
        script:
    	'''
    	env.selectedDevice = lockableResource()
    	if (target_version=="8855"){
          return [ \'000013d74e1b8613\', \'000011b7cc5b8613\',\'000013542e1b8613\']
        }
        if (target_version=="8845"){
          return [\'000009764d966153\', \'00000a174e966153\']
        }
		if (target_version=="8865" && automaticDeviceSelection=="Yes"){
		  def device = selectedDevice
          return [$device]
        }
        if (target_version=="8865" && automaticDeviceSelection=="No"){
          return [\'0000100fcf10e313\',\'000010d64e90e313\',\'000010f40c90e313\',\'000010f5cf10e313\',\'000013356c90e313\',\'000012350f3ae313\',\'00001276ac90e313\',\'000012f46c90e313\',\'0000130e4c90e313\',\'000013d74e1b8613\']
        }
        '''])
	}
		options { disableConcurrentBuilds()
            lock(resource: "$device")
            timestamps()
            
        }
    stages{
        stage("Environment"){
            steps{
                script{
            sh'''
            echo "Environmnet Variables"
            export
            '''
                }
            }
        }
    stage('SUMD Reference') {
            steps {
                script {
				echo "current ${sumd_branch} used for SUMD"
				if (params.sumd_branch == 'main'){
                    sh '''
                        #!/bin/bash
                        if [ -d "$SUMD_MAIN_WORKSPACE" ]; then
                            echo "Workspace directory already exists at $WORKSPACE."
                            cd $SUMD_MAIN_WORKSPACE
                            git pull
                        else
                            echo "SUMD directory does not exist. Creating now..."
                            git clone --reference $SUMD_MAIN_REFERENCE "ssh://vg.mahendran@gerrit.sarc.samsung.com:29418/sumd" -b mx/2025 $SUMD_MAIN_WORKSPACE

                            if [ $? -eq 0 ]; then
                                echo "Workspace directory created successfully at $WORKSPACE."
                            else
                                echo "Failed to create workspace directory at $WORKSPACE."
                            fi
                        fi
                    '''
                }
				if (params.sumd_branch == 'mx/2025'){
                    sh '''
                        #!/bin/bash
                        if [ -d "$SUMD_MX2025_WORKSPACE" ]; then
                            echo "Workspace directory already exists at $WORKSPACE."
                            cd $SUMD_MX2025_WORKSPACE
                            git pull origin mx/2025 -f
                        else
                            echo "SUMD directory does not exist. Creating now..."
                            git clone --reference $SUMD_MX2025_REFERENCE "ssh://vg.mahendran@gerrit.sarc.samsung.com:29418/sumd" -b mx/2025 $SUMD_MX2025_WORKSPACE

                            if [ $? -eq 0 ]; then
                                echo "Workspace directory created successfully at $WORKSPACE."
                            else
                                echo "Failed to create workspace directory at $WORKSPACE."
                            fi
                        fi
                    '''
                }
				if (params.sumd_branch == 'mx/2024'){
                    sh '''
                        #!/bin/bash
                        if [ -d "$SUMD_MX2024_WORKSPACE" ]; then
                            echo "Workspace directory already exists at $WORKSPACE."
                            cd $SUMD_MX2024_WORKSPACE
                            git pull origin mx/2024 -f
                        else
                            echo "SUMD directory does not exist. Creating now..."
                            git clone --reference $SUMD_MX2024_REFERENCE "ssh://vg.mahendran@gerrit.sarc.samsung.com:29418/sumd" -b mx/2024 $SUMD_MX2024_WORKSPACE

                            if [ $? -eq 0 ]; then
                                echo "Workspace directory created successfully at $WORKSPACE."
                            else
                                echo "Failed to create workspace directory at $WORKSPACE."
                            fi
                        fi
                    '''
                }
            }
        }
	}
	
	stage('SUMD SOCL Reference') {
            steps {
                script {
				echo "current ${sumd_branch} used for SUMD"
				if (params.sumd_branch == 'main'){
                    sh '''
                        #!/bin/bash
                        if [ -d "$SUMD_SOCL_MAIN_WORKSPACE" ]; then
                            echo "Workspace directory already exists at $WORKSPACE."
                            cd $SUMD_SOCL_MAIN_WORKSPACE
                            git pull
                        else
                            echo "SUMD directory does not exist. Creating now..."
                            git clone --reference $SUMD_MAIN_REFERENCE "ssh://vg.mahendran@gerrit.sarc.samsung.com:29418/sumd" -b mx/2025 $SUMD_SOCL_MAIN_WORKSPACE

                            if [ $? -eq 0 ]; then
                                echo "Workspace directory created successfully at $WORKSPACE."
                            else
                                echo "Failed to create workspace directory at $WORKSPACE."
                            fi
                        fi
                    '''
                }
				if (params.sumd_branch == 'mx/2025'){
                    sh '''
                        #!/bin/bash
                        if [ -d "$SUMD_SOCL_MX2025_WORKSPACE" ]; then
                            echo "Workspace directory already exists at $WORKSPACE."
                            cd $SUMD_SOCL_MX2025_WORKSPACE
                            git pull origin mx/2025 -f
                        else
                            echo "SUMD directory does not exist. Creating now..."
                            git clone --reference $SUMD_MX2025_REFERENCE "ssh://vg.mahendran@gerrit.sarc.samsung.com:29418/sumd" -b mx/2025 $SUMD_SOCL_MX2025_WORKSPACE

                            if [ $? -eq 0 ]; then
                                echo "Workspace directory created successfully at $WORKSPACE."
                            else
                                echo "Failed to create workspace directory at $WORKSPACE."
                            fi
                        fi
                    '''
                }
				if (params.sumd_branch == 'mx/2024'){
                    sh '''
                        #!/bin/bash
                        if [ -d "$SUMD_SOCL_MX2024_WORKSPACE" ]; then
                            echo "Workspace directory already exists at $WORKSPACE."
                            cd $SUMD_SOCL_MX2024_WORKSPACE
                            git pull origin mx/2024 -f
                        else
                            echo "SUMD directory does not exist. Creating now..."
                            git clone --reference $SUMD_MX2024_REFERENCE "ssh://vg.mahendran@gerrit.sarc.samsung.com:29418/sumd" -b mx/2024 $SUMD_SOCL_MX2024_WORKSPACE

                            if [ $? -eq 0 ]; then
                                echo "Workspace directory created successfully at $WORKSPACE."
                            else
                                echo "Failed to create workspace directory at $WORKSPACE."
                            fi
                        fi
                    '''
                }
            }
        }
	}
	stage('ANGLE Reference'){
        steps{
            script{
				echo "current ${angle_branch} used for Angle"  
    			if (params.angle_branch == 'main'){
    				sh'''
    				#!/bin/bash
    				if [ -d "$ANGLE_MAIN_WORKSPACE" ]; then
    					echo "Workspace directory already exists at $WORKSPACE."
    					cd $ANGLE_MAIN_WORKSPACE
    					git pull origin main -f
    				else
    					echo "ANGLE directory does not exist. Creating now..."
    					git clone --reference $ANGLE_MAIN_REFERENCE "ssh://vg.mahendran@gerrit.sarc.samsung.com:29418/dev_angle" -b main $ANGLE_MAIN_WORKSPACE
    
    					if [ $? -eq 0 ]; then
    						echo "Workspace directory created successfully at $WORKSPACE."
    					else
    						echo "Failed to create workspace directory at $WORKSPACE."
    					fi
    				fi
    				'''
            }
    			if (params.angle_branch == 'release'){
    				sh'''
    				#!/bin/bash
    				if [ -d "$ANGLE_RELEASE_WORKSPACE" ]; then
    					echo "Workspace directory already exists at $WORKSPACE."
    					cd $ANGLE_RELEASE_WORKSPACE
    					git pull origin release -f
    				else
    					echo "ANGLE directory does not exist. Creating now..."
    					git clone --reference $ANGLE_RELEASE_REFERENCE "ssh://vg.mahendran@gerrit.sarc.samsung.com:29418/dev_angle" -b release $ANGLE_RELEASE_WORKSPACE
    
    					if [ $? -eq 0 ]; then
    						echo "Workspace directory created successfully at $WORKSPACE."
    					else
    						echo "Failed to create workspace directory at $WORKSPACE."
    					fi
    				fi
    				'''
                    }
                if (params.angle_branch == 'mx/2025'){
    				sh'''
    				#!/bin/bash
    				if [ -d "$ANGLE_MX2025_WORKSPACE" ]; then
    					echo "Workspace directory already exists at $WORKSPACE."
    					cd $ANGLE_MX2025_WORKSPACE
    					git pull origin mx/2025 -f
    				else
    					echo "ANGLE directory does not exist. Creating now..."
    					git clone --reference $ANGLE_MX2025_REFERENCE "ssh://vg.mahendran@gerrit.sarc.samsung.com:29418/dev_angle" -b mx/2025 $ANGLE_MX2025_WORKSPACE
    
    					if [ $? -eq 0 ]; then
    						echo "Workspace directory created successfully at $WORKSPACE."
    					else
    						echo "Failed to create workspace directory at $WORKSPACE."
    					fi
    				fi
    				'''
                        }
                    }
            }
            
        }
	stage('SUMD Custom Patchset'){
		steps{
			script{
				if (params.sumd_patch != ""){
				sh'''
				    #!/bin/bash
    				echo "SUMD Patchset - ${sumd_patch}"
					custom_patchset(){
							git fetch ssh://vg.mahendran@gerrit.sarc.samsung.com:29418/sumd ${sumd_patch} && git cherry-pick FETCH_HEAD
							
							if [ $? -eq 0 ]; then
								echo "Patch applied successful!"
							else
								echo "Patch application failed!"
							fi
							}
					    	case $sumd_branch in
    							"main")
    								echo "Applying - ${sumd_patch} on SUMD Branch - ${sumd_branch}"
									cd ${SUMD_MAIN_WORKSPACE}
									custom_patchset
    								;;
    							"mx/2024")
    								echo "Applying - ${sumd_patch} on SUMD Branch - ${sumd_branch}"
									cd ${SUMD_MX2024_WORKSPACE}
									custom_patchset
    								;;
    							"mx/2025")
    								echo "Applying - ${sumd_patch} on SUMD Branch - ${sumd_branch}"
									cd ${SUMD_MX2025_WORKSPACE}
									custom_patchset
    								;;
    							*)
    								echo "That's an Invalid choice!"
    								;;
    						esac
				'''
				}
				else{
					echo "The parameter is Empty Building on TOT on SUMD Branch - ${sumd_branch}"
				}
			}
		}
	}
		stage('SUMD SOCL Custom Patchset'){
		steps{
			script{
				if (params.sumd_patch != ""){
				sh'''
				    #!/bin/bash
    				echo "SUMD Patchset - ${sumd_patch}"
					custom_patchset(){
							git fetch ssh://vg.mahendran@gerrit.sarc.samsung.com:29418/sumd ${sumd_patch} && git cherry-pick FETCH_HEAD
							
							if [ $? -eq 0 ]; then
								echo "Patch applied successful!"
							else
								echo "Patch application failed!"
							fi
							}
					    	case $sumd_branch in
    							"main")
    								echo "Applying - ${sumd_patch} on SUMD Branch - ${sumd_branch}"
									cd ${SUMD_SOCL_MAIN_WORKSPACE}
									custom_patchset
    								;;
    							"mx/2024")
    								echo "Applying - ${sumd_patch} on SUMD Branch - ${sumd_branch}"
									cd ${SUMD_SOCL_MX2024_WORKSPACE}
									custom_patchset
    								;;
    							"mx/2025")
    								echo "Applying - ${sumd_patch} on SUMD Branch - ${sumd_branch}"
									cd ${SUMD_SOCL_MX2025_WORKSPACE}
									custom_patchset
    								;;
    							*)
    								echo "That's an Invalid choice!"
    								;;
    						esac
				'''
				}
				else{
					echo "The parameter is Empty Building on TOT on SUMD Branch - ${sumd_branch}"
				}
			}
		}
	}
	stage('Angle Custom Patchset'){
		steps{
			script{
				if (params.angle_patch != ""){
				sh'''
				    #!/bin/bash
    				echo "ANGLE Patchset - ${angle_patch}"
					custom_patchset(){
					
							echo "* initializes submodules and 3rd party dependencies *"
							python3 scripts/run.py --init
				
							echo "* Obtain ANGLEs 3rd party dependencies *"
				
							#If building a branch that does not track origin/main, check Init() to make sure that it references the correct branch
							python3 scripts/run.py --init --skip-submodule-init
							
							echo "Initial requirements are initialized and applying patches"
							
							cd angle/
							git fetch ssh://vg.mahendran@gerrit.sarc.samsung.com:29418/angle ${angle_patch} && git cherry-pick FETCH_HEAD
							
							if [ $? -eq 0 ]; then
								echo "Patch applied successful!"
							else
								echo "Patch application failed!"
							fi
							}
    						case $angle_branch in
    							"main")
    								echo "Applying - ${angle_patch} on ANGLE Branch - ${angle_branch}"
									cd ${ANGLE_MAIN_WORKSPACE}
									custom_patchset
    								;;
    							"release")
    								echo "Applying - ${angle_patch} on ANGLE Branch - ${angle_branch}"
									cd ${ANGLE_RELEASE_WORKSPACE}
									custom_patchset
    								;;
    							"mx/2025")
    								echo "Applying - ${angle_patch} on ANGLE Branch - ${angle_branch}"
									cd ${ANGLE_MX2025_WORKSPACE}
									custom_patchset
    								;;
    							*)
    								echo "That's an Invalid choice!"
    								;;
    						esac
				'''
				}
				else{
					echo "The parameter is Empty Building on TOT on ANGLE Branch - ${angle_branch}"
				}
			}
		}
	}
	stage('SUMD Build'){
		steps{
			script{
    				sh'''
        				#!/bin/bash
    						echo "SUMD Branch - ${sumd_branch}"
							sumd_build(){
									SUMD_HEAD_COMMIT=$(git rev-parse HEAD)
									echo "Building SUMD on Commit ID : ${SUMD_HEAD_COMMIT}"
									
									# SUMD Build Script
									python3 scripts/run.py --os android --build-type release --build --clean --verbose
									if [ $? -eq 0 ]; then
										echo "Compilation successful!"
									else
										echo "Compilation failed!"
									fi
							}
    						case $sumd_branch in
    							"main")
    								echo "Running on ToT ${sumd_patch} on SUMD Branch - ${sumd_branch}"
									cd ${SUMD_MAIN_WORKSPACE}
									sumd_build
    								;;
    							"mx/2024")
    								echo "Running on ToT ${sumd_patch} on SUMD Branch - ${sumd_branch}"
									cd ${SUMD_MX2024_WORKSPACE}
									sumd_build
    								;;
    							"mx/2025")
    								echo "Running on ToT ${sumd_patch} on SUMD Branch - ${sumd_branch}"
									cd ${SUMD_MX2025_WORKSPACE}
									sumd_build
    								;;
    							*)
    								echo "That's an Invalid choice!"
    								;;
    						esac
    				'''
				    }
			    }
		    }
	stage('Revert SUMD Patchset'){
		steps{
			script{
    				sh'''
        				#!/bin/bash
    						echo "SUMD Branch - ${sumd_branch}"
							sumd_revert(){
									git reset --hard origin/${sumd_branch}
											
									if [ $? -eq 0 ]; then
									   echo "Revert successful!"
									else
									   echo "Revert failed!"
									fi
							}
    						case $sumd_branch in
    							"main")
    								echo "Running on ToT ${sumd_patch} on SUMD Branch - ${sumd_branch}"
									cd ${SUMD_MAIN_WORKSPACE}
									sumd_revert
    								;;
    							"mx/2024")
    								echo "Running on ToT ${sumd_patch} on SUMD Branch - ${sumd_branch}"
									cd ${SUMD_MX2024_WORKSPACE}
									sumd_revert
    								;;
    							"mx/2025")
    								echo "Running on ToT ${sumd_patch} on SUMD Branch - ${sumd_branch}"
									cd ${SUMD_MX2025_WORKSPACE}
									sumd_revert
    								;;
    							*)
    								echo "That's an Invalid choice!"
    								;;
    						esac
    				'''
				        }
			        }
	            }
		stage('SUMD SOCL Build'){
		steps{
			script{
    				sh'''
        				#!/bin/bash
    						echo "SUMD Branch - ${sumd_branch}"
							sumd_build(){
									SUMD_HEAD_COMMIT=$(git rev-parse HEAD)
									echo "Building SUMD on Commit ID : ${SUMD_HEAD_COMMIT}"
									
									# SUMD SOCL Build Script
									python3 scripts/run.py --os android --build-socl --build-type release --clean --build --verbose
									if [ $? -eq 0 ]; then
										echo "Compilation successful!"
									else
										echo "Compilation failed!"
									fi
							}
    						case $sumd_branch in
    							"main")
    								echo "Running on ToT ${sumd_patch} on SUMD Branch - ${sumd_branch}"
									cd ${SUMD_SOCL_MAIN_WORKSPACE}
									sumd_build
    								;;
    							"mx/2024")
    								echo "Running on ToT ${sumd_patch} on SUMD Branch - ${sumd_branch}"
									cd ${SUMD_SOCL_MX2024_WORKSPACE}
									sumd_build
    								;;
    							"mx/2025")
    								echo "Running on ToT ${sumd_patch} on SUMD Branch - ${sumd_branch}"
									cd ${SUMD_SOCL_MX2025_WORKSPACE}
									sumd_build
    								;;
    							*)
    								echo "That's an Invalid choice!"
    								;;
    						esac
    				'''
				    }
			    }
		    }
	stage('Revert SUMD SOCL Patchset'){
		steps{
			script{
    				sh'''
        				#!/bin/bash
    						echo "SUMD Branch - ${sumd_branch}"
							sumd_revert(){
									git reset --hard origin/${sumd_branch}
											
									if [ $? -eq 0 ]; then
									   echo "Revert successful!"
									else
									   echo "Revert failed!"
									fi
							}
    						case $sumd_branch in
    							"main")
    								echo "Running on ToT ${sumd_patch} on SUMD Branch - ${sumd_branch}"
									cd ${SUMD_SOCL_MAIN_WORKSPACE}
									sumd_revert
    								;;
    							"mx/2024")
    								echo "Running on ToT ${sumd_patch} on SUMD Branch - ${sumd_branch}"
									cd ${SUMD_SOCL_MX2024_WORKSPACE}
									sumd_revert
    								;;
    							"mx/2025")
    								echo "Running on ToT ${sumd_patch} on SUMD Branch - ${sumd_branch}"
									cd ${SUMD_SOCL_MX2025_WORKSPACE}
									sumd_revert
    								;;
    							*)
    								echo "That's an Invalid choice!"
    								;;
    						esac
    				'''
				        }
			        }
	            }
	stage('ANGLE Build'){
		steps{
			script{
    				sh'''
        				#!/bin/bash
    						echo "ANGLE Branch - ${angle_branch}"
							angle_build(){
									DEV_ANGLE_HEAD_COMMIT=$(git rev-parse HEAD)
									echo "Building DEV ANGLE on Commit ID : ${DEV_ANGLE_HEAD_COMMIT}"
									
									DEV_HEAD_COMMIT=$(cd angle;git rev-parse HEAD)
									echo "Building ANGLE on Commit ID : ${DEV_ANGLE_HEAD_COMMIT}"
											
									# ANGLE Build Script
											
									#echo "* initializes submodules and 3rd party dependencies *"
									#python3 scripts/run.py --init
						
									#echo "* Obtain ANGLEs 3rd party dependencies *"
						
									#If building a branch that does not track origin/main, check Init() to make sure that it references the correct branch
									#python3 scripts/run.py --init --skip-submodule-init
									
									echo "*Compiling ANGLE :: env -u JAVA_TOOL_OPTIONS python3 scripts/run.py --build-android *"
									env -u JAVA_TOOL_OPTIONS python3 scripts/run.py --build-android		
											
									if [ $? -eq 0 ]; then
									   echo "Compilation successful!"
									else
									   echo "Compilation failed!"
									fi
							}
    						case $angle_branch in
    							"main")
    								echo "Running on ToT - ${angle_patch} on ANGLE Branch - ${angle_branch}"
									cd ${ANGLE_MAIN_WORKSPACE}
									angle_build
    								;;
    							"release")
    								echo "Running on ToT - ${angle_patch} on ANGLE Branch - ${angle_branch}"
									cd ${ANGLE_RELEASE_WORKSPACE}
									angle_build
    								;;
    							"mx/2025")
    								echo "Running on ToT - ${angle_patch} on ANGLE Branch - ${angle_branch}"
									cd ${ANGLE_MX2025_WORKSPACE}
									angle_build
    								;;
    							*)
    								echo "That's an Invalid choice!"
    								;;
    						esac
    				'''
				        }
			        }
	            }
	stage('Revert ANGLE Patchset'){
		steps{
			script{
    				sh'''
        				#!/bin/bash
    						echo "ANGLE Branch - ${angle_branch}"
							angle_revert(){
										
									cd angle
									git reset --hard origin/${angle_branch}
											
									if [ $? -eq 0 ]; then
									   echo "Revert successful!"
									else
									   echo "Revert failed!"
									fi
							}
    						case $angle_branch in
    							"main")
    								echo "Reverting - ${angle_patch} on ANGLE Branch - ${angle_branch}"
									cd ${ANGLE_MAIN_WORKSPACE}
									angle_revert
    								;;
    							"release")
    								echo "Reverting - ${angle_patch} on ANGLE Branch - ${angle_branch}"
									cd ${ANGLE_RELEASE_WORKSPACE}
									angle_revert
    								;;
    							"mx/2025")
    								echo "Reverting - ${angle_patch} on ANGLE Branch - ${angle_branch}"
									cd ${ANGLE_MX2025_WORKSPACE}
									angle_revert
    								;;
    							*)
    								echo "That's an Invalid choice!"
    								;;
    						esac
    				'''
				        }
			        }
	            }
    stage('Clone One-ring'){
        steps{
		  dir("$WORKSPACE"){
            script{
                sh'''
                    if [ -d "$ONE_WORKSPACE" ]; then
                        echo "Workspace directory already exists at $ONE_WORKSPACE."
						# Set the expected owner
						EXPECTED_OWNER="jenkins"

						# Set the expected group
						EXPECTED_GROUP="jenkins"

						# Get the current owner and group of the folder
						CURRENT_OWNER=$(stat -c "%U" "$ONE_WORKSPACE")
						CURRENT_GROUP=$(stat -c "%G" "$ONE_WORKSPACE")
						if [ "$CURRENT_OWNER" == "$EXPECTED_OWNER" ] && [ "$CURRENT_GROUP" == "$EXPECTED_GROUP" ]; then
						  echo "The owner and group of the folder are as expected."
						  rm -rf $ONE_WORKSPACE            
						  echo "one ring directory checking out to commit ID :$ONE_COMMIT_ID ..."
						  git clone "ssh://vg.mahendran@gerrit.sarc.samsung.com:29418/one_ring"
						  cd $ONE_WORKSPACE
						  git checkout d810c0d4a0a064007e44af589f137012fae6f20f
						  cp -rf $CTS_SCRIPT_PATH/one_openGL.sh $ONE_WORKSPACE
						else
						  echo "The owner and/or group of the folder do not match the expected values."
						  echo "Expected Owner: $EXPECTED_OWNER, Expected Group: $EXPECTED_GROUP"
						  echo "Current Owner: $CURRENT_OWNER, Current Group: $CURRENT_GROUP"
						  echo ssir@2025 | sudo -S chown -R jenkins:jenkins "$ONE_WORKSPACE"
						  rm -rf $ONE_WORKSPACE            
						  echo "one ring directory checking out to commit ID :$ONE_COMMIT_ID ..."
						  git clone "ssh://vg.mahendran@gerrit.sarc.samsung.com:29418/one_ring"
						  cd $ONE_WORKSPACE
						  git checkout d810c0d4a0a064007e44af589f137012fae6f20f
						  cp -rf $CTS_SCRIPT_PATH/one_openGL.sh $ONE_WORKSPACE
						fi


					else    
						git clone "ssh://vg.mahendran@gerrit.sarc.samsung.com:29418/one_ring"
						
						if [ $? -eq 0 ]; then
							echo "Workspace directory created successfully at $WORKSPACE."
						else
							echo "Failed to create workspace directory at $WORKSPACE."
						fi
						echo "one ring directory checking out to commit ID :$ONE_COMMIT_ID ..."
						cd $ONE_WORKSPACE
						git checkout d810c0d4a0a064007e44af589f137012fae6f20f
						cp -rf $CTS_SCRIPT_PATH/one_openGL.sh $ONE_WORKSPACE
						
					fi
					'''
					}
				}
			}			
        }
    stage('Commenting out files'){
        steps{
            dir("$ONE_WORKSPACE"){
            script{
                sh'''
                echo " Updating/checking package_artifact.py"
        
                File=${ONE_WORKSPACE}/src/py/ci/package_artifact.py
                  if grep -q "#await Jfrog().download_async(self.local_path, None, self._jfrog_properties)" "$File"; ##note the space after the string you are searching for
                       then
                      echo "String!!It's available"
                  else
                      sed -i 's/await Jfrog().download_async(self.local_path, None, self._jfrog_properties)/#await Jfrog().download_async(self.local_path, None, self._jfrog_properties)/g' ${ONE_WORKSPACE}/src/py/ci/package_artifact.py
                      echo "The file is Updated"
                  fi
              
                echo " Updating/checking network_site.py"
              
                NET_FILE=${ONE_WORKSPACE}/src/py/common/network_site.py
                     if grep -q "#return NetworkSite.G2 if -time.timezone > 0 else NetworkSite.SARC" "$NET_FILE"; ##note the space after the string you are searching for
                     then
                          echo "String!!It's available"
                     else
                          sed -i 's/return NetworkSite.G2 if -time.timezone > 0 else NetworkSite.SARC/#return NetworkSite.G2 if -time.timezone > 0 else NetworkSite.SARC/g' ${ONE_WORKSPACE}/src/py/common/network_site.py
                          echo "The file is Updated"
                     fi
                     if grep -q "return NetworkSite.SARC" "$NET_FILE"; ##note the space after the string you are searching for
                     then
                        echo "String!!It's available"
                     else
                        awk 'END {print "        return NetworkSite.SARC"}' ${ONE_WORKSPACE}/src/py/common/network_site.py >> ${ONE_WORKSPACE}/src/py/common/network_site.py
                        echo "The file is Updated"
                    fi
            
                '''
                        }
                    }
                }
            }
    stage('Copy SUMD Artifacts'){
    steps{
        script{
        sh'''
        					copy_sumd_artifact(){
								echo "Copying SUMD Artifacts"
								
								mkdir -p ${ONE_WORKSPACE}/_out/builds/sumd
								
								cp -rv $1/out/android-arm64-release/* ${ONE_WORKSPACE}/_out/builds/sumd/
								
								if [ $? -eq 0 ]; then
								  echo "Copy successful!"
								else
								  echo "Copy failed!"
								fi
							}
					    	case $sumd_branch in
    							"main")
    								echo "Copying artifact - ${SUMD_MAIN_WORKSPACE} on SUMD Branch - ${sumd_branch}"
									copy_sumd_artifact ${SUMD_MAIN_WORKSPACE}
    								;;
    							"mx/2024")
    								echo "Copying artifact - ${SUMD_MX2024_WORKSPACE} on SUMD Branch - ${sumd_branch}"
									copy_sumd_artifact ${SUMD_MX2024_WORKSPACE}
    								;;
    							"mx/2025")
    								echo "Copying artifact - ${SUMD_MX2025_WORKSPACE} on SUMD Branch - ${sumd_branch}"
									copy_sumd_artifact ${SUMD_MX2025_WORKSPACE}
    								;;
    							*)
    								echo "That's an Invalid choice!"
    								;;
    						esac
        '''
        }
    }
}
stage('Copy SUMD SOCL Artifacts'){
    steps{
        script{
        sh'''
        					copy_sumd_artifact(){
								echo "Copying SUMD SOCL Artifacts"
								mkdir -p ${ONE_WORKSPACE}/_out/builds/socl
								cp -rv $1/out/android-arm64-release/* ${ONE_WORKSPACE}/_out/builds/socl/
								
								if [ $? -eq 0 ]; then
								  echo "Copy successful!"
								else
								  echo "Copy failed!"
								fi
							}
					    	case $sumd_branch in
    							"main")
    								echo "Copying artifact - ${SUMD_MAIN_WORKSPACE} on SUMD Branch - ${sumd_branch}"
									copy_sumd_artifact ${SUMD_MAIN_WORKSPACE}
    								;;
    							"mx/2024")
    								echo "Copying artifact - ${SUMD_MX2024_WORKSPACE} on SUMD Branch - ${sumd_branch}"
									copy_sumd_artifact ${SUMD_MX2024_WORKSPACE}
    								;;
    							"mx/2025")
    								echo "Copying artifact - ${SUMD_MX2025_WORKSPACE} on SUMD Branch - ${sumd_branch}"
									copy_sumd_artifact ${SUMD_MX2025_WORKSPACE}
    								;;
    							*)
    								echo "That's an Invalid choice!"
    								;;
    						esac
        '''
        }
    }
}
    stage('Copy ANGLE Artifacts'){
    steps{
        script{
				sh'''
				
				#!/bin/bash
				copy_angle_artifact(){
				
				echo "Copying ANGLE Artifacts"
				
				mkdir -p ${ONE_WORKSPACE}/_out/builds/dev_angle
				
				cp -r $1/angle/out/android-release/lib.unstripped ${ONE_WORKSPACE}/_out/builds/dev_angle/
				
				cp $1/angle/out/android-release/libEGL_samsung.so ${ONE_WORKSPACE}/_out/builds/dev_angle/
				
				cp $1/angle/out/android-release/libGLESv2_samsung.so ${ONE_WORKSPACE}/_out/builds/dev_angle/
				
				cp $1/angle/out/android-release/libfeature_support_samsung.so ${ONE_WORKSPACE}/_out/builds/dev_angle/
				
				cp $1/angle/out/android-release/libangle_util.so ${ONE_WORKSPACE}/_out/builds/dev_angle/
				
				cp $1/angle/out/android-release/libGLESv1_CM_samsung.so ${ONE_WORKSPACE}/_out/builds/dev_angle/
				
				cp $1/angle/src/tests/angle_end2end_tests_expectations.txt ${ONE_WORKSPACE}/_out/builds/dev_angle/
				
				cp $1/angle/out/android-release/angle_white_box_tests_apk/angle_white_box_tests-debug.apk ${ONE_WORKSPACE}/_out/builds/dev_angle/
				
				cp $1/angle/out/android-release/angle_unittests_apk/angle_unittests-debug.apk ${ONE_WORKSPACE}/_out/builds/dev_angle/
				
				cp $1/angle/out/android-release/angle_end2end_tests_apk/angle_end2end_tests-debug.apk ${ONE_WORKSPACE}/_out/builds/dev_angle/
				
				cp $1/angle/out/android-release/angle_perftests_apk/angle_perftests-debug.apk ${ONE_WORKSPACE}/_out/builds/dev_angle/
				
				if [ $? -eq 0 ]; then
					echo "Copy successful!"
				else
					echo "Copy failed!"
				fi
				}
		    				case $angle_branch in
    							"main")
    								echo "Copying artifact - ${ANGLE_MAIN_WORKSPACE} on ANGLE Branch - ${angle_branch}"
									copy_angle_artifact ${ANGLE_MAIN_WORKSPACE}
    								;;
     							"release")
     								echo "Copying artifact - ${ANGLE_RELEASE_WORKSPACE} on ANGLE Branch - ${angle_branch}"
 									
 									copy_angle_artifact ${ANGLE_RELEASE_WORKSPACE}
     								;;
     							"mx/2025")
     								echo "Copying artifact - ${ANGLE_MX2025_WORKSPACE} on ANGLE Branch - ${angle_branch}"
 									copy_angle_artifact ${ANGLE_MX2025_WORKSPACE}
     								;;
     							*)
     								echo "That's an Invalid choice!"
     								;;
     						esac
 					        '''
                         }
                     }
                 }
 	stage('Copy Khronos CTS Artifacts'){
 			steps{
 				script{
 				sh'''
 				
 				#!/bin/bash
 				echo "Copying Khronos CTS Artifacts"
 				cp -rv $PACKAGES_PATH/glcts ${ONE_WORKSPACE}/_out/builds
 				
 				if [ $? -eq 0 ]; then
 					echo "Copy successful!"
 				else
 					echo "Copy failed!"
 				fi
 				
 				'''
 					}
 				}
 			}
 	stage('Copy OCL_ICD binaries'){
 			steps{
 				script{
 				sh'''
 				
 				#!/bin/bash
 				echo "Copying OCL_ICD binaries"
 				mkdir -p ${ONE_WORKSPACE}/_out/builds/ocl_icd
 				
 				cp -rv $PACKAGES_PATH/ocl_icd/* ${ONE_WORKSPACE}/_out/builds/ocl_icd/
 				
 				if [ $? -eq 0 ]; then
 					echo "Copy successful!"
 				else
 					echo "Copy failed!"
 				fi
 				'''
 					}
 				}
 			}
		stage('BSP Verification'){
		    steps {  
				dir("$BSP_SCRIPT_PATH"){			
                    script{ 
				if (params.BSP_PACKAGE != ''){						
							sh'''
							if [ -d "$BSP_PACKAGES_PATH/$target_version/$android_version/$BSP_PACKAGE" ]; then
								echo "$BSP_PACKAGE already exists at $BSP_PACKAGES_PATH/$target_version/$android_version."
							else
									build_id=$(echo ${BSP_PACKAGE} | cut -d "_" -f 1 | cut -c 2-)
									
									./fetch_bsp.sh "${target_version}" "${android_version}" "${BSP_PACKAGES_PATH}/${target_version}/${android_version}" "${image_type}" "${build_id}"
        							
									if [ $? -eq 0 ]; then
        								echo "BSP Fetching successful!"
        							else
        								echo "BSP Fetching failed!"
        							fi
        					fi
							'''
                            }
				if (params.BSP_PACKAGE == ''){
								sh'''
										./fetch_bsp.sh "${target_version}" "${android_version}" "${BSP_PACKAGES_PATH}/${target_version}/${android_version}" "${image_type}" "${build_id}"
									    
										if [ $? -eq 0 ]; then
        								    echo "BSP Fetching successful!"
        							    else
        								    echo "BSP Fetching failed!"
        							    fi									
			
								'''
		                 }
					}
				}
			}
			}
		stage('Flash Custom BSP device'){
			agent {
				node('ubuntu-17')
            } 
		    steps {           
    		    dir("$DEVICE_FLASH_SCRIPT_PATH"){
                    script{ 
						
						if (params.BSP_PACKAGE == ''){
                            env.BSP_PACKAGE = sh(returnStdout: true, script: 'cat ${BSP_PACKAGES_PATH}/${target_version}/${android_version}/bsp_info.txt').trim()
							sh'''
				            ./deviceflash_script.sh "${target_version}" "${android_version}" "${BSP_PACKAGE}" "${device}"
							if [ $? -eq 0 ]; then
								echo "BSP Flashing successful!"
							else
								echo "BSP Flashing failed!"
							fi
							'''
                            }
						if (params.BSP_PACKAGE != ''){
                            //env.BSP_PACKAGE = sh(returnStdout: true, script: 'cat ${BSP_PACKAGES_PATH}/${target_version}/${android_version}/bsp_info.txt').trim()
							sh'''
				            ./deviceflash_script.sh "${target_version}" "${android_version}" "${BSP_PACKAGE}" "${device}"
							if [ $? -eq 0 ]; then
								echo "BSP Flashing successful!"
							else
								echo "BSP Flashing failed!"
							fi
							'''
                            }
		                 }
                    }
					}
		}
	stage('OpenGL CTS Execution'){
		    agent {
        node('ubuntu-17')
            } 
        steps{
            dir("$ONE_WORKSPACE"){
                        catchError(buildResult: 'FAILURE', stageResult: 'FAILURE')
        {
        script{
                sh'''
					sshpass -p ssir@2025 ssh -o StrictHostKeyChecking=no -t vg.mahendran@107.108.213.17 -t "cd /var/lib/jenkins/workspace/openGL_cts_custom;source /var/lib/jenkins/sarc-bashrc;echo ssir@2025 | sudo -S chown -R vg.mahendran:vg.mahendran one_ring;cd one_ring;export ANDROID_SERIAL=${device};export PATH=/usr/local/android-sdk/platform-tools:${PATH};./one_openGL.sh ${device} ${test_type} ${sumd_branch} ${angle_branch} ${target_version} ${android_version}"
    			'''
                            }
                         }
        }
    
                    }
                }
    stage('Publish CTS Report'){
        steps{
            publishHTML(
                target: [
                    reportDir: '/var/lib/jenkins/workspace/openGL_cts_custom/one_ring/_ci_out/reports/.',
                    reportFiles: 'summary.html',
                    reportName: 'CTS Test Report for openGL'
                    ]
                    )
             }
	  }
	stage('OpenGL CTS Report Archive'){
        	steps{
        		script{
        		def time_stamp = sh(returnStdout: true, script: 'date "+%m-%d-%Y_%H_%M_%S"').trim()
        		def testReportDir = "${USERCONTENT_DIR}/${BUILD_NUMBER}_${target_version}_${JOB_NAME}_REPORT_${time_stamp}"
                env.TEST_REPORT_DIR = testReportDir
                echo "TEST_REPORT_DIR: ${TEST_REPORT_DIR}"
        		sh'''
        		mkdir -p $TEST_REPORT_DIR
        		# Check if the hidden ".sam-dir" folder exists in the project directory
        
        		if [ -d "$REPORT_DIR" ]; then
        			# Copy the contents of "report" to the workspace directory
        			cp -r "${REPORT_DIR}/"* "${TEST_REPORT_DIR}/"
        			echo "Operation successful: Copied $REPORT_DIR contents to ${TEST_REPORT_DIR} directory"
        		else
        			echo "Operation failed: $REPORT_DIR not found in project directory"
        		fi
        		'''

				def reportPath = "${TEST_REPORT_DIR}"
				def reportURL = reportPath.replace("/var/lib/jenkins", "http://107.108.213.7:8080")
				finalreportURL = reportURL.replace("'", "")
        		    }
        	    }
            }
		stage('Device recovery'){
				agent {
				node('ubuntu-17')
            } 
		    steps {           
    		    dir("$RECOVER_SCRIPT_PATH"){
                    script{ 
							sh'''			    
							./recover_device.sh "${device}"							
							if [ $? -eq 0 ]; then
								echo "Device Recovery successful!"
							else
								echo "Device Recovery  failed!"
							fi
							'''
                            }
		                 }
                    }
		}
        }
    post { 
        always { 
            echo 'Reverting oner ring ownership'
            sh'''sshpass -p ssir@2025 ssh -o StrictHostKeyChecking=no -t vg.mahendran@107.108.213.17 -t "cd /var/lib/jenkins/workspace/openGL_cts_custom;echo ssir@2025 | sudo -S chown -R jenkins:jenkins one_ring"'''
            echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
            
            echo "Please find the the link to access the Report directory ${finalreportURL}"
            
            echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
        }
    }
}
