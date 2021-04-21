def tag = ""
def image = ""
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
                    	echo "preparing to deploy ${env.BRANCH_NAME}"
                        echo "get docker image tag from values file"
                        tag = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()

                        echo "get harbor image sha from harbor"
                        image = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
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
                // container('git') {
                //     sh "echo Validating deployment..."
                //     sh "echo ${image}"
                //     sh "apk add jq"
                //     sh """
				// 		wget -O- -q \
				// 			--post-data='{
				// 				"resourceUri": "harbor.rode.lead.prod.liatr.io/rode-demo/rode-demo-node-app@${image}"
				// 			}' \
				// 			--header='Content-Type: application/json' \
				// 			'http://rode.rode-demo.svc.cluster.local:50051/v1alpha1/policies/a6bb1c3c-376b-4e4a-9fa4-a88c27afe0df:attest' | jq .pass | grep true
                //     """
                // }
                container('git') {
                	echo "validate deployment"
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
                    sh "helm upgrade -f env-values.yaml -f environments/${env.BRANCH_NAME}/values.yaml --install demo-app-test charts/demo-app -n rode-demo-app"
                }
            }
        }
    }
}
