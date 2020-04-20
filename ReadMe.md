This folder contains the helm charts for Policy Server and Access Gateway

***************************************************************************************************************************************
Steps for deploying policyserver components:
This folder contains the helm charts for PolicyServer.

Server-Components configurations can be found in  deploy/cloud-native/helm/server-components/values.yaml

Values.yaml is broadly categorized into three parts
1. Global Values
    Contains the common values for the Siteminder Administrative PolicyServer and Policyserver PODS.

2. Administrative Policy server
   Contains the settings specific to Administrative Policy server POD

3. Policy server
    Contains the settings specific to Policy server POD



Helm Deployment
***************

Deploy Administrative Policy Server
===================================
1. Navigate to deploy/cloud-native/helm/server-components/ folder
2. Create Namespace if not already present
3. Run the following command to deploy only Administrative Policy Server
   
    helm install <release name> . -n <namespace_name> --set admin.enabled=true --set policyServer.enabled=false
    
    Ex:-
    helm install adminps . -n cassoserver --set admin.enabled=true --set policyServer.enabled=false

4. For checking helm deployment use "helm ls -n <namespace_name>", for pods "kubectl get pods -n <namespace_name>"
5. To delete , use "helm ls -n <namespace_name>" for getting list of releases and apply "helm delete <release_name,release_name1,..> -n <namespace_name>"

Deploy Policy Server
====================
1. Navigate to deploy/cloud-native/helm/server-components/ folder
2. Create Namespace if not already present
3. Run the following command to deploy only Policy Server
   helm install <release name> . -n <namespace_name> --set admin.enabled=false --set policyServer.enabled=true
   
   Ex:-
   helm install ps . -n cassoserver --set admin.enabled=false --set policyServer.enabled=true

4. For checking helm deployment use "helm ls -n <namespace_name>", for pods "kubectl get pods -n <namespace_name>"
5. To delete , use "helm ls -n <namespace_name>" for getting list of releases and apply "helm delete <release_name,release_name1,..> -n <namespace_name>"

Deploy both Administrative Policy Server and PolicyServer
=========================================================
1. Navigate to deploy/cloud-native/helm/server-components/ folder
2. Create Namespace if not already present
3. Run the following command to deploy only Policy Server
   helm install <release name> . -n <namespace_name> --set admin.enabled=true --set policyServer.enabled=true

   Ex:-
   helm install casso . -n cassoserver --set admin.enabled=true --set policyServer.enabled=true

4. For checking helm deployment use "helm ls -n <namespace_name>", for pods "kubectl get pods -n <namespace_name>"
5. To delete , use "helm ls -n <namespace_name>" for getting list of releases and apply "helm delete <release_name,release_name1,..> -n <namespace_name>"


Helm Upgrade
************

Upgrade Only Administrative Policy Server
=========================================
1. Navigate to deploy/cloud-native/helm/server-components/ folder
2. Update the corresponding parameters in values.yaml
3. Run the following command to upgrade only Administrative Policy Server 
  
   helm upgrade <release name> . -n <namespace_name> --set admin.enabled=true --set policyServer.enabled=false

   Ex:-
   helm upgrade adminps . -n cassoserver --set admin.enabled=false --set policyServer.enabled=true

Upgrade Only Policy Server
==========================
1. Navigate to deploy/cloud-native/helm/server-components/ folder
2. Update the corresponding parameters in values.yaml
3. Run the following command to upgrade only Policy Server
   helm upgrade <release name> . -n <namespace_name> --set admin.enabled=false --set policyServer.enabled=true
   Ex:- 
   helm upgrade ps . -n cassoserver --set admin.enabled=false --set policyServer.enabled=true
   
Upgrade both Administrative Policy Server and  Policy Server
============================================================
1. Navigate to deploy/cloud-native/helm/server-components/ folder
2. Update the corresponding parameters in values.yaml
3. Run the following command 
   helm upgrade <release name> . -n <namespace_name> --set admin.enabled=true --set policyServer.enabled=true
   Ex:-
   helm upgrade sso . -n cassoserver --set admin.enabled=true --set policyServer.enabled=true   

   Note: sso is the realese name where both Administrative Policy Server and  Policy Server are deployed.

Helm Rollback
*************

1. If something goes wrong after successful upgrade you can rollback to previous revision using "helm rollback <component_release_name> <revision> -n <namespace_name>", here revision should be last successful revision.
2. Getting revision of release name use: "helm history <release_name> -n <namespace_name>"
3. Execption if image version is wrongly typed in values.yaml:-
	case 1: If image version is wrongly given in values.yaml of adminps, 
			execute: helm history <release_name_of_adminps> -n <namespace_name>
					 helm rollback <release_name_of_adminps> <last_successful_revision> -n <namespace_name>
					 kubectl delete pod <casso-admin-<last_oridinal_indexed_number>> -n <namespace_name>
			it will revert back to last successful image version, after this you can type image version value correctly and follow uprade procedure.
	case 2: If image version is wrongly given in values.yaml of ps, it will take lot of time to end meanwhile check for pods whether they are giving "ImagePullBackOff error", if they are giving it.
			execute: helm history <release_name_of_ps> -n <namespace_name> 
					 helm rollback <release_name_of_ps> <last_successful_revision> -n <namespace_name>

Helm Packaging
**************

HELM packaging helps you in re-using your initial HELM configuration across other servers without having to download the packages from the SSO repository onto each server in your environment and
then configuring HELM charts on each server.
The process of re-using HELM configuration involves the following steps at a high-level:
1. Download the required TAR packages from the SSO repository onto your initial server.
2. Untar the packages and configure the required HELM charts using values.yaml files.
3. Create a package for the configured HELM charts.
4. Upload the package onto another server.	

This topic describes the steps in HELM packaging.

Prerequisites
1. Set up the required HTTP servers that must act as HELM repositories and create a folder on the servers where the package from SiteMinder must be placed.
   Example Folder Name on HTTP Server: helmpackages
2. Download the required packages from the SSO repository.
3. Configure the values.yaml files of siteminder server-Components consisting of settings related to Administrator Policy Server and Policy Server as required.

Create HELM Package on a Server and Download it and deploy on Another Server
============================================================================

Follow these steps:
1. Create a package for HELM charts on a server.
   a) Navigate to the folder(/cloud-native/helm/server-components/) where the Chart.yaml is present
   b) Run the below command
      > helm package . 
	  
      A TAR file is generated for each Chart.yaml file.
   b) Move the TAR file that is generated in Step 1 into a new folder.
      Example Folder Name: ssotarpackages

2. Generate an index file
   a) Navigate to new folder(ssotarpackages).
   b) Run the below command.
      > "helm repo index [new_folder_name] --url [httpserver_url_till_foldername_which_will_store_tgzfiles]"
      Index.yaml file will be generated.	
	  	  
3. Upload the package onto the folder that is created on HTTP Server to store files from
SiteMinder.
   a. Copy the TAR files that are generated in Step 1a and the index.yaml file that is generated in
Step 2b into the folder created on HTTP server as part of the prerequisites.

4. Add the HELM repository of HTTP server onto a local system.
   a. Log onto the server where you want to deploy SiteMinder.
   b. Run the following command to add the HELM repository of the HTTP server:
      helm repo add [name_of_repo] [httpserver_url_till_foldername_that_stores_the_copied_tgz_and_index_files]
      
	  Example: helm repo add sso http://httpserverhostname/helmpackages
      The HELM repository that was saved on the HTTP server is downloaded onto the local system. The repository is named as specified in the command.
   c. View the charts within the repository using the following command
       helm search repo [name_of_repo]
	   
	   Example: helm search repo sso
       Example Chart Format: sso/Server-Components
	   
5. Deploy the server-Components from repository
   a) Search for repository
       helm search repo [name_of_repo]
       Example: helm search repo sso
   b) Deploy siteminder server-components
      a) Deploy both Administrative Policy Server and Policy Server
		 helm install <release name> [name_of_repo]/server-components -n <namespace_name> --set admin.enabled=true --set policyServer.enabled=true 
		 
		 Example: 
		 helm install sso sso/server-components -n cassoserver --set admin.enabled=true --set policyServer.enabled=true
		 
	  b) Deploy Policy Server
		 helm install <release name> [name_of_repo]/server-components -n <namespace_name> --set admin.enabled=false --set policyServer.enabled=true 
		 
		 Example: 
		 helm install ps sso/server-components -n cassoserver --set admin.enabled=false --set policyServer.enabled=true
		 
      c) Deploy Administrative Policy Server
		 helm install <release name> [name_of_repo]/server-components -n <namespace_name> --set admin.enabled=true --set policyServer.enabled=false
		 
		 Example: 
		 helm install adminps sso/server-components -n cassoserver --set admin.enabled=true --set policyServer.enabled=false	 
 	   
    c) Check the status of deployment

        helm ls -n <namespace_name>



***************************************************************************************************************************************


Steps for deploying Access Gateway component:
deploy/cloud-native/helm/access-gateway folder contains the helm charts for Access Gateway.

Access Gateway
   deploy/cloud-native/helm/access-gateway/Chart.yaml - Helm chart for Access Gateway Component
   deploy/cloud-native/helm/access-gateway/values.yaml - Contains the values specific to access-gateway
   deploy/cloud-native/helm/access-gateway/templates - Contains the kubernetes templates for access-gateway


Steps to deploy Access Gateway 
----------------------------------------

Copy access-gateway folder along with its contents to any directory

Method 1 :
1) Generate the values.yaml with default values

 > helm show  values ./access-gateway > sm_values.yaml

2) Now edit the sm_values.yaml as per required configuration.

3) Run the below command for installing the access-gateway 
   
   Here smagv1 is the application name i.e accessgateway name ( Refer helm install --help  for more details )
     > helm install smagv1 -f sm_values.yaml --namespace cassoag ./access-gateway
	 
	 
 
      OR for installing with debug option (this will show the computed values) use the below command
	  
	 > helm install smagv1 --debug -f sm_values.yaml --namespace cassoag ./access-gateway

4) Command for displaying the list helm charts 
     > helm ls -n cassoag 
 
 
 


Method 2 : (Edit the default values directly in the yaml)
1) Edit the values.yaml files (access-gateway/values.yaml) as per customer configuration.
2) Run the below command for installing the access-gateway 

     > helm install smagv1 --namespace cassoag ./access-gateway
 
     OR for installing with debug option (this will show the computed values) use the below command
	 > helm install smagv1 --debug --namespace cassoag ./access-gateway



Steps to Upgrade Access Gateway 
----------------------------------------
Copy contents of access-gateway folder to any directory

Identify the helm release name that needs to be upgraded
 
1) Update the user specific values by above Method 1 (Step 1 and 2) OR Method 2 (Step 1)

2) Run the below command to list the helm releases nad make a note of the access-gateway release name.

  > helm ls -n cassoag 

3) Run the below command to upgrade (pass the release-name obtained form the above step.)
   > helm upgrade -f sm_values.yaml <release-name> -n cassoag  ./access-gateway 
   
   Based on your requirements, if time required to pull the images take longer time then explicitly add timeout value during upgrade process
   > helm upgrade -f sm_values.yaml <release-name> -n cassoag  --timeout <timeout value> ./access-gateway 

Steps to Rollback Access Gateway 
----------------------------------------
Copy contents of siteminder folder to any directory

Identify the helm release name that needs to be upgraded
 
1) Run the below command to list the helm releases nad make a note of the access-gateway release name.

  > helm ls  -n cassoag 

2) Run the below command to find the list revisons available for access-gateway release
   > helm history <release-name> -n cassoag 
   
3) Run the below command to rollback to specific revison.
   > helm rollback <release-name> <revison-number> -n cassoag 






 
