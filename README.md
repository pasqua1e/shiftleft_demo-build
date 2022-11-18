### How to setup the Shift Left demo story with all stop-gaps

## TLDR;
1) Create a new 'secret text' in Jenkins, called PCCS_API (the value needs to be your Access Key in Prisma Cloud, which is in the format AccessKey::SecretKey)
2) Create a new pipeline in Jenkins and point to this repo
3) Make sure you also install the ANSI Color plugin in Jenkins (it fixes the formatting and looks also nicer)

## Detailes steps:
You can clone this repo and put it in the demo-build gogs server, so that can in turn be used in the demo-build Jenkins server with all pre-existing vars and passwors.
You can also point the demo-build server to this repo, once it becomes public, or to your fork, instead of copying to the gogs server.

Until this is committed to the demo-build source code and then its own Gogs and Jenkins servers, you'll need to clone this repo's content (into your demo-build gogs server, if you wish) and then add a new pipeline in your Jenkins server based on that.


Link to running this demo: https://drive.google.com/file/d/1fFPoteVm61hqeQOb40Uzhdxw-DKDkLtG/view

PC_USER,PC_PASS and PC_CONSOLE are pre populated from the yaml file from the central demo build. 

### If you're using the demo-build env for this, then:
1. Clone this repo (either into the gogs server in demo-build or your own Github), so all permissions are re-used. If you just point the Jenkins new pipeline to this repo, make sure you add an auth key to GitHub.
2. Add a new project in the Jenkins demo-build server and point its configuration to the `Jenkinsfile` that's in this repo you just cloned into gogs.
3. All auth keys are already part of the demo-build env and the config files here, except the PCSS_API, which needs to be added.


### Stop-gaps to set up before a demo:
1. In the evilpetclinic repo (i.e. in se-cloud-emea https://github.com/se-cloud-emea/evil.petclinic) verify that the pom.xml file has the `jackson-databind` dependency un-commented and with a vulnerable version i.e. 2.10.2
2. Defend > Vulnerabilities > Code repositories: add the evilpetclinic repo (i.e. in se-cloud-emea: https://github.com/se-cloud-emea/evil.petclinic to scan it
3. Defend > Vulnerabilities Images > CI: set up a rule that scopes evilpetclinic (or all) to fail on High vulnerabilities or above.
4. Defend > Compliance > Containers and images > CI: set up a rule that scopes evilpetclinic (or all) to fail on malware.

Above stop gaps will fail these steps in the Jenkins pipeline:
1. Git repo scan
2. Image scan

