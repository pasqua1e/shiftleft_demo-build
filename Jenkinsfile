node {
    files= ['deploy.yml']
    options {
        ansiColor('xterm')
    }
    stage('Clone repository') {
        checkout scm
    }


    stage('Download latest twistcli') {
        withCredentials([usernamePassword(credentialsId: 'twistlock_creds', passwordVariable: 'TL_PASS', usernameVariable: 'TL_USER')]) {
            sh 'curl -k -v -u $TL_USER:$TL_PASS --output ./twistcli https://$TL_CONSOLE/api/v22.01/util/twistcli'
            sh 'sudo chmod a+x ./twistcli'
        }
    }
	
    stage('Check image Git dependencies has no vulnerabilities') {
        try {
            withCredentials([usernamePassword(credentialsId: 'twistlock_creds', passwordVariable: 'TL_PASS', usernameVariable: 'TL_USER')]) {
                sh('chmod +x files/checkGit.sh && ./files/checkGit.sh')
            }
        } catch (err) {
            echo err.getMessage()
            echo "Error detected"
			throw RuntimeException("Build failed for some specific reason!")
        }
    }

    
    stage('Apply security policies (Policy-as-Code) for evilpetclinic') {
        withCredentials([usernamePassword(credentialsId: 'twistlock_creds', passwordVariable: 'TL_PASS', usernameVariable: 'TL_USER')]) {
            sh('chmod +x files/addPolicies.sh && ./files/addPolicies.sh')
        }
    }

    

    stage('Scan image with twistcli') {
        try {
		sh 'docker pull pasqu4le/evilpetclinic:latest'
            withCredentials([usernamePassword(credentialsId: 'twistlock_creds', passwordVariable: 'TL_PASS', usernameVariable: 'TL_USER')]) {
                sh 'curl -k -u $TL_USER:$TL_PASS --output ./twistcli https://$TL_CONSOLE/api/v1/util/twistcli'
                sh 'sudo chmod a+x ./twistcli'
                sh "./twistcli images scan --u $TL_USER --p $TL_PASS --address https://$TL_CONSOLE --details pasqu4le/evilpetclinic"
            }
        } catch (err) {
            echo err.getMessage()
            echo "Error detected"
			throw RuntimeException("Build failed for some specific reason!")
        }
    }


    stage('Scan K8s yaml manifest with Bridgecrew') {  
	withDockerContainer(image: 'kennethreitz/pipenv:latest', args: '-u root --privileged -v /var/run/docker.sock:/var/run/docker.sock' ) {
		withCredentials([string(credentialsId: 'PCCS_API', variable: 'PCCS_API')]) { 
			
			script { 
                    		sh """export PRISMA_API_URL=https://api.prismacloud.io
                    		pipenv install
                    		pipenv run pip install bridgecrew
                    		pipenv run bridgecrew -s -o cli --directory . --bc-api-key $PCCS_API --repo-id jenkins/$BUILD_TAG"""
                	}
		}
	}
    }
	

    stage('Deploy evilpetclinic') {
        sh 'kubectl create ns evil --dry-run -o yaml | kubectl apply -f -'
        sh 'kubectl delete --ignore-not-found=true -f files/deploy.yml -n evil'
        sh 'kubectl apply -f files/deploy.yml -n evil'
        sh 'sleep 30'
    }

    stage('Run bad Runtime attacks') {
        sh('chmod +x ./files/runtime_attacks.sh && ./files/runtime_attacks.sh')
    }

    stage('Run bad HTTP stuff for WAAS to catch') {
        sh('chmod +x ./files/waas_attacks.sh && ./files/waas_attacks.sh')
    }
}
