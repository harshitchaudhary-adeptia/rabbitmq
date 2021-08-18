#!/usr/bin/env groovy
def ver = BUILD_NUMBER
def getEnvVar(String paramName){
    return sh (script: "grep '${paramName}' ENV_VARS/project.properties|cut -d'=' -f2", returnStdout: true).trim();
}
pipeline
{
    agent {
  	    label 'LinuxAgent'
    }
    stages {
	    stage('Init'){
        	steps{
        		script{
					env.LATEST_TAG=env.LATEST_TAG
        			        env.BASE_DIR = pwd()
					env.COMMON_FILE = getEnvVar('COMMON_FILE')
        			        env.PROJECT_NAME=getEnvVar('PROJECT_NAME')
					env.NEXUS_VERSION=getEnvVar('NEXUS_VERSION')
					env.NEXUS_PROTOCOL=getEnvVar('NEXUS_PROTOCOL')
					env.NEXUS_URL=getEnvVar('NEXUS_URL')
					env.NEXUS_MAVEN_REPOSITORY=getEnvVar('NEXUS_MAVEN_REPOSITORY')
                	                env.NEXUS_CREDENTIAL_ID=getEnvVar('NEXUS_CREDENTIAL_ID')
					env.NEXUS_DOCKER_REPOSITORY=getEnvVar('NEXUS_DOCKER_REPOSITORY')
					env.DOCKER_IMAGE_URL=getEnvVar('DOCKER_IMAGE_URL')
					env.NEXUS_DOCKER_CREDENTIALS_ID=getEnvVar('NEXUS_DOCKER_CREDENTIALS_ID')
					env.NEXUS_DOCKER_REGISTRY_URL=getEnvVar('NEXUS_DOCKER_REGISTRY_URL')
					env.NEXUS_HELM_REPO_NAME=getEnvVar('NEXUS_HELM_REPO_NAME')
                	                env.NEXUS_HELM_REPO_CREDENTIAL_ID=getEnvVar('NEXUS_HELM_REPO_CREDENTIAL_ID')
					env.NEXUS_HELM_REPO_URL=getEnvVar('NEXUS_HELM_REPO_URL')
					env.HELM_RELEASE_NAME=getEnvVar('PROJECT_NAME')
					env.HELM_CHART_VERSION=getEnvVar('HELM_CHART_VERSION')
				        env.NAMESPACE=getEnvVar('NAMESPACE')
				        env.WHITESOURCE_ORGTOKEN=getEnvVar('WHITESOURCE_ORGTOKEN')
					env.CLUSTER_CONTEXT=getEnvVar('CLUSTER_CONTEXT')
					env.ENVIRONMENT=getEnvVar('ENVIRONMENT')
				}
			}
        }

	    stage('Docker Build'){
            steps {
                		echo 'Building...'
                	script {
                     		try {
					                print "*** building docker image ***"
					                withCredentials([usernamePassword(credentialsId: "${NEXUS_DOCKER_CREDENTIALS_ID}", passwordVariable: 'password', usernameVariable: 'username')])
						        {
                         			sh '''
										#!/bin/sh
					 					LATEST_TAG_REV=$(git rev-list --tags --max-count=1)
					 					LATEST_COMMIT_REV=$(git rev-list HEAD --max-count=1)
					 					if [ -n "$LATEST_TAG_REV" ]; then
    					 				LATEST_TAG=$(git describe --tags "$(git rev-list --tags --max-count=1)")
					 					fi
					 					if [ "$LATEST_TAG_REV" != "$LATEST_COMMIT_REV" ]; then
					 					LATEST_TAG=$(git rev-list HEAD --max-count=1 --abbrev-commit)
    					 				echo "$LATEST_TAG"
					 					fi
                            		                                echo "${password} | docker login -u ${username} ${NEXUS_DOCKER_REGISTRY_URL} --password-stdin"
										docker build -t ${DOCKER_IMAGE_URL}/${PROJECT_NAME}:${LATEST_TAG} 3.8/ubuntu/Dockerfile
										docker push ${DOCKER_IMAGE_URL}/${PROJECT_NAME}:${LATEST_TAG}
										aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 716848636303.dkr.ecr.us-east-2.amazonaws.com
										docker tag ${DOCKER_IMAGE_URL}/${PROJECT_NAME}:${LATEST_TAG} 716848636303.dkr.ecr.us-east-2.amazonaws.com/adeptia/${PROJECT_NAME}:${LATEST_TAG}
										docker push 716848636303.dkr.ecr.us-east-2.amazonaws.com/adeptia/${PROJECT_NAME}:${LATEST_TAG}
			 		            		'''
                     					}
					           }
					            catch(err) {
						            error(err.message)
                        			print "*** Error while building/pushing image ***"
                    			}
                		}
          		   }
       		}
		stage('publish Helm to nexus'){
            steps {
                script {
                     	withCredentials([usernamePassword(credentialsId: "${NEXUS_HELM_REPO_CREDENTIAL_ID}", passwordVariable: 'password', usernameVariable: 'username')]){
                        sh '''
                            helm repo add  nexus https://${username}:${password}@${NEXUS_HELM_REPO_URL}
	                    helm package deployment/${PROJECT_NAME}/
			    curl -u ${username}:${password} https://${NEXUS_HELM_REPO_URL} --upload-file ${PROJECT_NAME}-${HELM_CHART_VERSION}.tgz -v
                         	'''
                     	}
                }
           }
        }
	}
}
