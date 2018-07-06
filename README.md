HTML PUBLIC "-//W3C//DTD HTML 4.0//EN"       1.0 Online Help for Query Service   ######  Online Help for Query Service

  Query Services Help contains the following topics:

  *  [See What is Query Service?.](QAASonlineHelp.htm#83538)
 *  [See Prerequisites.](QAASonlineHelp.htm#80949)
 *  [See Sign Up screen.](QAASonlineHelp.htm#43722)
 *  [See Set Up VPC Peering.](QAASonlineHelp.htm#68551)
 *  [See After Setting Up VPC Peering.](QAASonlineHelp.htm#61342)
 *  [See Important Things to Know.](QAASonlineHelp.htm#30989)
   ######  What is Query Service?

  Query Service is a cost-effective way to achieve scalability in the cloud, that allows you to respond to varying query workload demand without having to size your environments to peak demand. This elastic query capacity is provided as a fully managed service, eliminating the need for setting up and managing the underlying infrastructure required to scale capacity. You can better control costs, reduce the time to implement a solution, and provide a better experience to end users.

  MarkLogic Query Service works by attaching MarkLogic e-nodes to your existing MarkLogic cluster and group and auto-scale depending on your workload based on contract limits. These e-nodes are stateless and hold no user data. 

  Query Service adds dynamic query capacity to an existing cluster - it does not provide any data persistence. You must supply your own database cluster running in an AWS account in order to utilize the Query Service. MarkLogic Query Service includes the MarkLogic license for the service; it does not include the licenses for the database cluster you are using with the Query Service. You cannot bring your own licenses to the Query Service. Query Service is compatible with the MarkLogic Essential Enterprise as well as the MarkLogic Global Enterprise licenses. You can utilize any options you have licensed across your database cluster. Use of the MarkLogic Query Service requires a subscription that includes a service agreement.

  As part of the MarkLogic service:

  *  MarkLogic provisions and manages instances by the e-nodes
 *  Provisions and manages other infrastructure used by the e-nodes, such as VPC, EBS, etc.
 *  Performs patching of the OS and MarkLogic server used in the e-nodes part of the service
 *  Supports the service 
   ######  What is not included in the service?

  *  Query Service does not provide any data services as customers are fully responsible for setting up and managing their d-nodes and database. 
 *  Since there is no data stored in the e-nodes that are part of the service, there is no backup
 *  There will be no patching of the OS and MarkLogic server used in the d-nodes. MarkLogic provides the provisioning and management of instances used by the e-nodes, VPC, VPC peering.
     ######  Prerequisites

  MarkLogic Query Service is only available in AWS. In order to use Query Service:

  *  You must have an existing MarkLogic cluster in an AWS VPC, with an existing database, group, and appserver to which you want to attach the service. We recommend configuring a separate group and appserver for Query Service. When you create the group, it will have no host assigned to it. Query Service will assign its hosts. 
 *  You must be running MarkLogic 9.0-6 or later.
 *  You must configure VPC peering, in order for the MarkLogic e-nodes to see the existing cluster d-nodes on the customer side.
 *  Telemetry (config, meters, logs) must be turned on for your cluster.
 *  Encryption must be turned on for your logs.
    ######  Sign Up screen

  The first step is to sign up with Amazon Web Services. Click on XX to open the Sign Up screen. Enter the information for your account into the fields on this screen. If you already have an account, click the * Sign In* link. Fields marked with an asterisk (*) are required. 

  ![](images/signupScreen.gif)  

  *  You must click the checkbox next to the Terms of Use agreement to start up the service. 
   The Sign Up screen contains these fields:

     Field

    Value

      First Name*

    Enter your first name

      Last Name*

    Enter your last name

      Username*

    Enter your AWS username

      Email*

    Enter your email address

      Password*

    Enter a password for this account. Re-enter your password in the following field.

      Help

    Click the “* ?”* for more information and help about passwords.

      Country*

    From the drop-down menu, select the country in which this cluster is located.

      I agree to Terms of Use and Privacy Policy

    Click the check box to accept the Terms of Use. Click the * Terms of Use* link for the details of the Terms of Use or the * Privacy Policy* link to view the Privacy Policy document. This box must be checked in order to sign up. 

      Sign Up

    Click Sign Up to continue.

      

   ######  Set Up VPC Peering

  You need to enable VPC peering so that the Query Service can connect with your cluster. To do this, you will need to prepare your MarkLogic cluster on AWS. After that, you will need to configure VPC, then configure your network, and finally create the service. 

  ![](QAASonlineHelp-2.gif)  

  To set up and enable VPC peering, follow these steps: 

  ######  Prepare Cluster

  The Query Service needs to be able to join the user’s cluster in order to provide the requested e-nodes, so that the clusters can “see” one another. A user with an existing VPC cluster creates a peer role for Query Service so the Query Service can join the cluster. MarkLogic provides a cloud formation template to create the necessary AIM role. There are two things you need to do: enable trace events and and create the peer-role for the MarkLogic Query Service. These two things can be done in any order.

  To enable trace events use the * set-trace-event.xqy* function. Run this function on your MarkLogic instance/cluster on S3. Run the following code in Query Console on your MarkLogic server. For example:

  xquery version "1.0-ml";  
 import module namespace admin = "http://marklogic.com/xdmp/admin"   
   at "/MarkLogic/admin.xqy";  
   
 let $config := admin:get-configuration()  
 let $groupid := admin:group-get-id($config, "[UseYourOwnGroup]")  
 let $config := admin:group-set-trace-events-activated($config, $groupid, fn:true())  
 return admin:save-configuration($config);  
   
   
 xquery version "1.0-ml";  
 import module namespace admin = "http://marklogic.com/xdmp/admin"   
 at "/MarkLogic/admin.xqy";  
   
 let $config := admin:get-configuration()  
 let $groupid := admin:group-get-id($config, "[UseYourOwnGroup]")  
 let $event := admin:group-trace-event("Allow Dynamic Hosts")  
 let $config := admin:group-add-trace-event($config, $groupid, $event)  
 let $event := admin:group-trace-event("Allow Dynamic Host Connections")  
 let $config := admin:group-add-trace-event($config, $groupid, $event)  
 return admin:save-configuration($config);

   

  You can also enable trace events and allow dynamic hosts using the MarkLogic Admin UI. Under Groups, <Group Name>, Diagnostics, click on the Configure tab. 

  ![](QAASonlineHelp-3.gif)  

  To create the peer role for the MarkLogic Query Service, use * the peer-role-template* that can be found in the S3 marketplace (* <https://s3-us-west-2.amazonaws.com/marklogic-services-resources/templates/peer-role.template>* ). Use the template when you create the new stack. 

  ![](images/createPeerRole.gif)  

  When you fill out the template, use the MarkLogic Service ID that was generated when you signed up. Enter the VPC ID for the cluster you want to use. 

  If you used the MarkLogic template (mlcluster.vpc.template) to create your cluster, you can find the VPC ID in the stack labeled “Create a VPC for MarkLogic cluster” under the Resources tab. It will be named * *-VpcStack-** automatically generated by AWS. The VPC ID will start with * vpc-** .

  If you used the other template to launch MarkLogic into your existing VPC cluster, use the same VPC ID that you used with that template (the VPC ID starting with * vpc-** ).

  Click Next. 

  ![](images/ReviewPeerRole.gif)  

  Review the details that you entered. You can edit them if necessary by clicking Previous. If everything looks correct, click Create.

  *  Write down the ARN that is generated by this step. You will need this information in the next step. 
    ######  Configure VPC

  This step creates the actual peering between the clusters. To accomplish this, you need to provide the ARN (a unique resource identifier) to MarkLogic Query Services so that the service has a role and the access necessary to perform the peering. You will need to provide the AWS account ID as part of this step. 

   

  ![](images/ConfigVPC.gif)  

  To configure your VPC, enter your VPC information into these fields. Fields marked with an asterisk (*) are required. 

     Field

    Value

    Where to find this information

      VPC ID*

    Starts with * vpc-* 

     

      AWS Account ID*

     

    * [https://docs.aws.amazon.com/IAM/latest/UserGuide/console\_account-alias.html](https://docs.aws.amazon.com/IAM/latest/UserGuide/console_account-alias.html)* 

      Peer Role ARN*

    Example:

  arn:aws:iam::123456789012:role/MLAAS-PeerRole-peerRole-PUE2MD0KEMI2

    * <https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html#arn-syntax-iam>* 

      E-Nodes VPC CIDR

     

     

      E-Nodes Subnet 1 CIDR

     

     

      E-Nodes Subnet 2 CIDR

     

     

      D-Nodes Subnet CIDRs*

     

     

      Region*

    The selected region must be in the same as the region as the one the d-node cluster resides in 

     

      Configure/Cancel

    Click Configure to configure VPC or Cancel the process.

     

     *  The selected region must be the same as the one the d-node cluster resides in.
   When VPC Peering is complete you will see the Configured VPC screen:

  ![](images/configuredVPC.gif)  

  Copy the Peering Connection ID from this screen. You will need this information in the next step 

   ######  Configure Network

  MarkLogic provides a template on S3 (* configure-network.template* ) that can be found on the S3 marketplace (* <https://s3-us-west-2.amazonaws.com/marklogic-services-resources/templates/configure-network.template>* ) to configure route table, and subnets for Query Service. Enter the information for this to specify a subnet for the service, with a unique IP in the network. This step sets up the networking for the service. 

  To do this, use the template to create another stack. 

  ![](images/createVPCstack.gif)  

  To configure your VPC stack, enter your VPC parameters into these fields. Fields marked with an asterisk (*) are required. 

     Field

    Value

    Where to find this information

      Route Table ID

    Example:

  * rtb-92b140e9* 

     

      Security Group ID

    Example:

  * sg-fcf5108c* 

     

      Dynamic E-node Subnet CIDR 1

    Example:

  * 10.1.0.0/25* 

    Provide the same CIDR block information that you used in the previous step.

      Dynamic E-node Subnet CIDR 2

    Example:

  * 10.1.1.0/27* 

    Provide the same CIDR block information that you used in the previous step.

      VPC Peering ID

    Example:

  * pcx-06a13011c03ca54ab* 

    Copy the VPC Peering Connection ID from the Configured VPC screen in the previous step. 

     When you have entered the information, click Next. 

  ![](images/ReviewVPCstack.gif)  

  Review the information. If you need to change anything, click Previous to edit the fields. If everything looks correct, click Create. 

  Once the VPC is configured the new stack is displayed. The status should be CREATE\_COMPLETE in the list of stacks on the AWS portal 

   ######  Create Service

  This step defines the Query Service, specifies the service type (the number of dynamic e-nodes). 

  ![](images/CreateService.gif)  

  Enter information for the new services into the fields on this screen. Fields marked with an asterisk (*) are required.

   

     Field

    Value

    Where to find this information

      Name*

    The name of the service

    Choose a relevant name for your service. 

      Service Type*

     * qs.e15.small* 

    Choose the size of the deployment. Options are qs.e15.small, qs.e15.medium, qs.e15.large.

  The default value is qse15.small.

      Region*

    * us-west-2* 

    This value come from the form in step 2.

      Cluster ID*

    * 10460249393463477218* 

    Enter the cluster ID for the service in order to retrieve Groups, MarkLogic version, App Servers, and port information.

  Click * Get config* to retrieve configuration information. 

       Group*

    * Default* 

    Select the group from the drop-down menu. 

      App Server*

    App-Services (8000)

    Select the App Server for this service from the drop-down menu.

      Version*

    9000600

    This field is auto-populated from the cluster ID information.

      D-Node Hostnames*

    ip-10-0-1-106.us-west-2.compute.internal,ip-10-0-65-218.us-west-2.compute.internal,ip-10-0-32-130.us-west-2.compute.internal

    This field is auto-populated from the cluster ID information.

      Create/Cancel

    Click Create to create the new service, or click Cancel to cancel the process.

     

     Review the information you entered. If it looks correct, click Create.

    ######  After Setting Up VPC Peering

  Once the VPC peering is set up, your next step is to sign in and start using Query Service. You will see the dashboard screen, showing the existing services. Click Create Service to add a new service.

  ######  Dashboard

  Once you have signed in, the dashboard is displayed. These values were selected at the time of service configuration. 

  (placeholder graphic)

  ![](images/dashboard.gif)  

  The dashboard displays a view of the current registered services, with the following columns. 

    

     Field

    Value

      #

    Number ID for this service

      Service

    The name of the service

      App Server

    The App Server on which the service is running

      Cluster

    The cluster ID for the cluster that contains the app server

      Service Type

    The size of the current deployment. Options are qs.e15.small, qs.e15.medium, qs.e15.large.

      Region

    The region select for the hosting of this service from the drop-down menu in the Configure VPC form.

      Group

    This is populated with the current group. 

      Status

    This can be “CREATING”, “CREATED”, “CREATE\_FAILED”, “DELETING”, “DELETED”, DELETE\_FAILED”

      Configure

    Click the icon to edit this service

      ######  Edit

  The Edit screen contains the following fields; the fields cannot be edited. You can click either Terminate or Cancel on this screen. 

     Field

    Value

      Name

    The name of the service. This is already populated, but can be edited.

      Service Type

    Choose the size of the deployment. Options are qs.e15.small, qs.e15.medium, qs.e15.large. 

      Status

    This can be “CREATING”, “CREATED”, “CREATE\_FAILED”, “DELETING”, “DELETED”, DELETE\_FAILED”

      Region

    Select or change the region for the hosting of this service from the drop-down menu.

      Cluster ID

    This is populated with the current cluster ID for the service

      Group

    This is populated with the current group. 

      App Server

    This is populated with the current App Server name, but you can select a different App Server from the drop-down list. 

      AppServer Port

    Selected when the service is created.

      Version

    The version of MarkLogic that is running in this cluster.

      D-Node Hostnames

    Specified when the service is created. 

      D-Node Subnet CIDR

    Specified when the service is created. 

      Terminate /Cancel

    Click * Terminate* to delete the existing service, or click * Cancel* to cancel the process.

       ######  Important Things to Know

  Check the items in this section to ensure that Query Service will work correctly.

  ######  D-node Hostnames Need to be Resolvable

  For the Query Service to function correctly, the MarkLogic d-node hostnames need to be resolvable. 

   ######  Trace Event Required

  For a group to allow a dynamic e-node to join the group, “Allow Dynamic Hosts” should be set on that group.

  For hosts in a group to accept connections/request from a dynamic e-node, “Allow Dynamic Host Connections” must be allowed on that group. 

  You could partition your application such that hosts in a group serve an application, and only that application/group allows e-nodes to join them dynamically. And the joining e-nodes could belong to a different group (for instance, where all dynamic e-nodes are grouped together).

   ######  Deleting VPC Peering

  You can only delete VPC peering if you have the service in a “deleted” state. Possible states are “created”, “creating”, “terminating”, and “deleted”. If your service is in any state except the “deleted” state and you attempt to delete VPC peering, you will need to call Support. 

      
