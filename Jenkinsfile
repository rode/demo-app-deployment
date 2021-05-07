def tag = ""
def image = ""
def build_result = 'SUCCESS'
if (env.BRANCH_NAME == 'staging' || env.BRANCH_NAME == 'prod') {
   build_result = 'FAILURE'
} else {
   build_result = 'UNSTABLE'
}
pipeline {
    agent {
        kubernetes {
            yamlFile "jenkins-agent.yaml"
        }
    }

    stages {
    	stage('Collect info') {
            steps {
                container('git') {
                    script {
                        sh "apk add yq"
                    	echo "preparing to deploy ${env.BRANCH_NAME}"
                        echo "get docker image tag from values file"
                        tag = sh(script: "yq r env-values.yaml image.tag", returnStdout: true).trim()
                    }
                }
			}
    	}
        stage("Evaluate Policy"){
        	when {
        		anyOf {
					expression {
						env.CHANGE_TARGET != null && (env.CHANGE_TARGET == 'staging' || env.CHANGE_TARGET == 'prod')
					}
    				branch 'dev';
    				branch 'staging';
    				branch 'prod';
        	  	}
        	}
            steps {
                container('git') {
                    sh "echo Validating deployment..."
                    sh "echo ${tag}"
                    sh "apk add jq"
                    script {
						try {
							sh """
								wget -O- -q \
									--post-data='{
										"resourceUri": "harbor.internal.lead.prod.liatr.io/rode-demo/rode-demo-node-app@sha256:${tag}"
									}' \
									--header='Content-Type: application/json' \
									'https://rode.internal.lead.prod.liatr.io/v1alpha1/policies/4127d475-80ec-4d36-9ece-98029176bdec:attest' | jq .pass | grep true
							"""
						} catch (err) {
							if (env.BRANCH_NAME == 'staging' || env.BRANCH_NAME == 'prod' || env.CHANGE_TARGET == 'staging' || env.CHANGE_TARGET == 'prod') {
							   build_result = 'FAILURE'
							   sh "exit 1"
							} else {
							   currentBuild.result = "UNSTABLE"
							}
						}
                    }
                }
            }
        }
        stage('Deploy') {
    		when {
    			anyOf {
    				branch 'dev';
    				branch 'staging';
    				branch 'prod';
    			}
			}
            steps {
                 container('helm') {
                    sh "helm version"
                    sh "helm upgrade -f env-values.yaml -f environments/${env.BRANCH_NAME}/values.yaml --install demo-app-test charts/demo-app -n rode-demo-app-${env.BRANCH_NAME}"
                }
            }
        }
    }
}
