pipeline {
	agent any
	options {
		ansiColor('xterm')
	}
	environment {
    PRISMA_API_URL='https://api.prismacloud.io'
}
stages {
    


	    
    stage('Clone repository') {
	    steps {
        checkout scm
	    }
    }


    stage('Download latest twistcli') {
	    steps {
        withCredentials([usernamePassword(credentialsId: 'twistlock_creds', passwordVariable: 'TL_PASS', usernameVariable: 'TL_USER')]) {
            sh 'curl -k -v -u $TL_USER:$TL_PASS --output ./twistcli https://$TL_CONSOLE/api/v22.01/util/twistcli'
            sh 'sudo chmod a+x ./twistcli'
        }
	    }
    }
	
    stage('Check image Git dependencies has no vulnerabilities') {
	    steps {
        
            withCredentials([usernamePassword(credentialsId: 'twistlock_creds', passwordVariable: 'TL_PASS', usernameVariable: 'TL_USER')]) {
                sh('chmod +x files/checkGit.sh && ./files/checkGit.sh')
           
       		 }
	    }
    }

    
    stage('Apply security policies (Policy-as-Code) for evilpetclinic') {
	    steps {
        withCredentials([usernamePassword(credentialsId: 'twistlock_creds', passwordVariable: 'TL_PASS', usernameVariable: 'TL_USER')]) {
            sh('chmod +x files/addPolicies.sh && ./files/addPolicies.sh')
        }
	    }
    }

    

    stage('Scan image with twistcli') {
	    steps {
        
		sh 'docker pull pasqu4le/evilpetclinic:latest'
            withCredentials([usernamePassword(credentialsId: 'twistlock_creds', passwordVariable: 'TL_PASS', usernameVariable: 'TL_USER')]) {
                sh 'curl -k -u $TL_USER:$TL_PASS --output ./twistcli https://$TL_CONSOLE/api/v1/util/twistcli'
                sh 'sudo chmod a+x ./twistcli'
                sh "./twistcli images scan --u $TL_USER --p $TL_PASS --address https://$TL_CONSOLE --details pasqu4le/evilpetclinic"
            }
        
	    }
    }


    stage('Scan K8s yaml manifest with Bridgecrew') {  
	steps {
		withCredentials([string(credentialsId: 'PCCS_API', variable: 'PCCS_API')]) { 	
			script { 
				docker.image('bridgecrew/checkov:latest').inside("--entrypoint=''") {
					sh 'checkov -s -d . --use-enforcement-rules -o cli --bc-api-key $PCCS_API --repo-id jenkins/$BUILD_TAG'
				}
                    	}
		}
	
	    }
    }
	

    stage('Deploy evilpetclinic') {
	    steps {
        sh 'kubectl create ns evil --dry-run -o yaml | kubectl apply -f -'
        sh 'kubectl delete --ignore-not-found=true -f files/deploy.yml -n evil'
        sh 'kubectl apply -f files/deploy.yml -n evil'
        sh 'sleep 30'
	    }
    }

    stage('Run bad Runtime attacks') {
	    steps {
        sh('chmod +x ./files/runtime_attacks.sh && ./files/runtime_attacks.sh')
	    }
	   }

    stage('Run bad HTTP stuff for WAAS to catch') {
	    steps {
        sh('chmod +x ./files/waas_attacks.sh && ./files/waas_attacks.sh')
    	}
    }
    
}
}
