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
        stage("evaluate policy"){
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

                    catchError(buildResult: "$build_result", stageResult: 'FAILURE') {
						sh """
							wget -O- -q \
								--post-data='{
									"resourceUri": "harbor.rode.lead.prod.liatr.io/rode-demo/rode-demo-node-app@sha256:${tag}"
								}' \
								--header='Content-Type: application/json' \
								'http://rode.rode-demo.svc.cluster.local:50051/v1alpha1/policies/a6bb1c3c-376b-4e4a-9fa4-a88c27afe0df:attest' | jq .pass | grep true
						"""
					}
                    script {
                    	sh "env"
                    }

                }
            }
        }
        stage('deploy') {
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
