
RBAC rules
================

Run this as a cluster-admin.

   kubectl create -f rbac/roles.yaml



Weblogic server creation steps
=======================================

This is to deploy Weblogic application Server. Weblogic is used to deploy enterprise JAVA applications.
Weblogic should contain below servers:

   1. Admin Server  
   2. managed servers
   
   Managed servers can be 0 or many and in production,managed servers are clustered and applications are deployed to managed servers. managed servers are tightly coupled with Admin Server.
   Admin Server must be started in order to start Managed servers.
   
   Below are the steps to create a Seblogic domain using Kubernetes.


   
1. Create a local storage
 
       kubectl create -f weblogic/storage.yml
	   
2. Create Persistant Volume.

       kubectl create -f weblogic/persistant-volume.yml
	   
	   Note: recomended approach for multinode architechture for Weblogic is to use NFS. Used storage a slocal NFS. provided path:  "/refresh/user_projects/domains/weblogic"

3. Create Volume claim.

        kubectl create -f weblogic/claim-persistant.yml


		

4. Create Admin Server.

       kubectl create -f weblogic/adminServer.yaml
	
5. Copy kan.war,deployApplication.py,deployApplication.sh in the host volume directory.
      The path should be path/applications.  The files wil be automatically copied to /u01/oracle/user_projects/applications
	  
	  This is the recommmended approach to deploy application in Weblogic.
	   
	   Check the admin-server POD logs until,it is started. In the log it should record "RUNNING" state.
	   
6. Create a service/Nodeport for AdminServer:

        kubectl create -f weblogic/adminServer-service.yaml
		
		Nodeport is used as we need to adinister Weblogic using Admin Console.Console is UI tool,to manage Weblogic domain and it is deployed automatically in Admin Server.
		
		
7. Use " kubectl get pods -o wide "	to check,where the adminServer pod is orchestrated. Use http://node-hosname:32001/console in the browser.

          Credentials: weblogic/welcome1		
		  
		
8. Create Managed server and Services.

      kubectl create -f weblogic/managedServers.yaml
	  
	  StatefulSet is used as ,to scale  up and down the managed servers. 

		
	   

8. Scale Managed server.

    kubectl scale statefulsets managed-server --replicas=3
	
	Manual deletion of Weblogic server is required after scaling down. It should be deleted befor scaling up,if managed server is scaled down before.
	

10. Copy kan.war file in your Volume Path/applications.

     Note: The image was created by me earlier for a POC . Deployment script is not created yet.
	 
11. Deploy the application to the cluster of managed servers . The context root is "/kan" .

      ==> Login to Weblogic console as per step 6.
	  ==> In the left pane,click "Deployments"
	  ==> Click "Locka nd Edit" in the left pane.
	  ==> Click on "Install" on the right pane.
	  ==> Click on "user_projects" from the patch and then click "applications" .
	  ==> Selct the radio button "kan.war"  and click Next.
	  ==> Select "Install this deployment as an application" and click Next .
	  ==> Select "Cluster1 ==>  All servers in the cluster". Click next and click "Finish".
	  ==> Click on "Activate Changes" on left pane.
	  ==> On the right pane,click "Control" tab.
	  ==> Select "kan" .Click on "STart  ==> Servicing ALL Requests  ==> yes" .
	  


	 Managed serevrs ports are ClusterIP,it will not be accessible in the browser. To test it "kubectl get ep managed-server" and use "curl http://ip:8001/kan"
	 
	 
Ingress steps
===============


   1. Create ingress rules to route request to Managed servers.
   
       kubectl create -f ingress/ingress.yaml
	   
	   
   2. Create ingress controller.
   
        kubectl create -f ingress/ingress-controller.yml
		
   3.  Create a default service,when service will not be avalaible,it will go to default service.
   
         kubectl create -f echoservice.yml
		 
	4. To verify ingress:

           kubectl get ingress	
		   
	5. Ingress Nodeport is 31614.
	
	    Access http://hostname:31614/kan
		
		


      









