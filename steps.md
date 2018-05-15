# Lab Exercise 2

### -----DELETE------
>**Please note that the images used in the lab guide are representative and NOT based on any specific pod. Please use the information in the lab guide instead.**

Lab-2 will be used to demonstrate L4-L7 service insertion in managed mode with device manager to simulate an enterprise network and/or cloud provider’s application delivery offering while allowing the application owner to manage the L4-L7 device application template via a centralized device manager. 

We will use the F5 BIG-IP VE Virtual ADC and F5 iWorkflow workflow management tool to demonstrate this functionality.

Different from the previous CiscoLive, we will demonstrate the Policy-Based Redirect feature that was introduced in APIC version 2.0 in this lab exercise.

### ------DELETE------

## ++++++ Payal suggestion ADD ++++++++
Lab-2 will use Ansible to configure BIG-IP to correspond to the Unmanaged mode of APIC deployment.
- Goal is to perform L2-L3 stitching between the Cisco ACI fabric and F5 BIG-IP.
- Configure the L4-L7 parameters on F5 BIG-IP using ansible playbooks.

### +++++++++ ADD +++++++++

![Environment](../img/env-lab2.png)

### --------DELETE ---------

## Verify the F5 BIG-IP iApps

F5 iApps is a user-customized framework for deploying application, providing a flexible way to automate tasks and templatize functionality on F5 devices.

Log into the F5 BIG-IP **{TBIGIPIP}** with the following username and password from the web browser:  
BIG-IP: **[https://{TBIGIPIP}](https://{TBIGIPIP})**  
Username: **admin**  
Password: **cisco123**  

After you have logged into the F5 BIG-IP GUI, you should see the following:

![Big IP GUI](../img/bigipgui.png)

In the Navigation pane, click the **iApps -> Templates**. You should see the iApp template – _appsvcs\_integration\_v1.0\_001_ pre-loaded into the F5 BIG-IP:

![iApps Template](../img/iapptemp.png)

>iApps to be used by iWorkflow / APIC integration must be exist in BIG-IP in order for iWorkflow to be discovered.

##Set up the F5 iWorkflow Cloud Management

Log into the F5 iWorkflow **{TBIGIQIP}** with the following username and password from the web browser:  
iWorkflow: **[https://{TBIGIQIP}](https://{TBIGIQIP})**  
Username: **admin**  
Password: **cisco123**  
	
After you have logged into the F5 iWorkflow UI, you should see the iWorkflow UI similar to the following:

![iWorkflow GUI](../img/bigiqgui.png)

If the UI is not in the **Clouds and Services** configuration menu, click on the icon **Clouds and Services** and the UI should switch to the Clouds and Services configuration menu:

![Devices](../img/iqdevice.png)

After switching to the Clouds and Services configuration menu, you should see the following:

![Cloud](../img/cloud.png)

At this point, you are ready to register the F5 BIG-IP with the F5 iWorkflow. In order to do this, you will need to move the mouse to the left or right side of the screen (the iWorkflow has a dynamic interface) and then the Cloud Devices menu should appear. Select **Devices**:

![Cloud Devices](../img/cloud-device.png)

The menu will shift the menu to the left or right to allow the Cloud Devices menu to appear.

![Cloud Devices 1](../img/devices.png)

Move the mouse over to **Devices** menu, click the **+** and **Discover Device** to register the F5 BIG-IP with your F5 iWorkflow:

![Discover Device](../img/disc-dev.png)

Once you are in the **Devices** menu, you can register the F5 BIG-IP by using the BIG-IP’s IP address and credential as the following:  

IP Address: **{TBIGIPIP}**  
Username: **admin**  
Password: **cisco123**  
	
Click **Save** to register the BIP IP device:

![Big IP](../img/iqip.png)

Wait approximately 2 minutes and double click the registered BIG-IP and verify its status. It should say **_Available_** when the BIG-IP is registered with the iWorkflow:

![Big IP Details](../img/ipdetails.png)

After BIG-IP is successfully discovered by iWorkflow, the iApp resides on BIG-IP are now exposed to iWorkflow. You can now create a template based on iApps.

Move your mouse to the left or right side of the screen and allow the Cloud **Catalog** menu to appear.

Next, select **Catalog**. When the Catalog menu appears on the screen, click **+** to continue:

![Catalog](../img/catalog.png)

A New Template screen will appear as the following:

![New Template](../img/newtemp.png)

Enter and select the following in the New Template:  
Name: **{TSTUDENT}-http-no-persistence**  
Input Parameters: **All Options**  
Application Type: **appsvcs\_integration\_v1.0\_001**  

![Select Catalog](../img/selectcatalog.png)  

You can now edit all the available options that need to be included with this template.  

Expand the _**Virtual Server Listener & Pool Configuration**_ by clicking the **>**. Scroll down and **CHECK** the following to make them **Tenant Editable**. What this does is allow the parameters supported by the iWorkflow device package to expose the parameters to APIC. If desired, we can limit what is exposed via a custom device package (this reduces the complexity). It is highly recommended to expose only what is needed to APIC:

**pool__addr**  
**pool__port**

![IQ Config](../img/iqconfig.png)

Expand the **pool__Members** by clicking the **>** to allow additional items to be editable in APIC. Scroll down and **CHECK** the following to make them **Tenant Editable**:  
**Port**

![Port](../img/port.png)

Scroll down and expand the _**Virtual Server Configuration**_ by clicking **>** to remove the default settings of the following to **EMPTY** (Tenant Editable is already **UNCHECK**):

**vs__ProfileDefaultPersist**  
**vs__ProfileFallbackPersist**

![IQ 1](../img/iq-1.png)

>This is just used for our lab scenario to show the BIG-IP will load share all the connections between the Web servers. For a real life deployment scenario, you should always consult with the application owner before editing.

Scroll down a bit more and **CHECK** the following to make it **Tenant Editable**. In addition, remove the default setting of the following to **EMPTY**:

**vs__SNATConfig**

![IQ 2](../img/iq-2.png)

We will use the above setting to demonstrate policy-based redirect with ACI fabric in the later section of this lab.

Click **Save** to complete the template:

![Save Template](../img/save.png)

Now, the newly created device package template is ready to be consumed by the APIC. The next step is to create the Cloud connectors which will generate a custom device package using this template as an APIC service function. Move your mouse to the left or right side of the screen to make the **Clouds** menu to appear

![Connectors](../img/connectors.png)

To create a new Cloud connector, move the mouse to the **Clouds** menu and the **+** should appear.

![New Connector](../img/newconn.png)

Click the **+** to enter a new Cloud connector. Enter and select the following to complete the creation of the Cloud connector:  
Name: **{TSTUDENT}**  
Connector Type: **Cisco APIC**  

![Selection](../img/conn-select.png)

Click **Save** to finish.

![Save Connector](../img/saveconn.png)

Now, move your mouse to the Cloud connectors menu and double click the **{TSTUDENT}** cloud connector. At this point, the **{TSTUDENT}** connector should expand and it should allow you to save the custom device package to your remote desktop.

![Download device package](../img/downloaddevpkg.png)

Click the Download Device Package **F5DevicePackage.zip** to save the custom device package to your desktop.

##Import the Custom Device Package

Switch to your APIC GUI and click the following to upload the device package:  
**L4-L7 Services -> Packages -> L4-L7 Service Device Type**  
	
Click the **ACTIONS** button at the Work pane and choose **IMPORT DEVICE PACKAGE**  

![Import Device Package](../img/importdevpkg.png)

A new pop-up should appear to allow you to choose the device package to be installed:

![Browse](../img/browse.png)

Click **BROWSE** and choose the previous downloaded device package - **F5DevicePackage.zip** from your desktop and click **SUBMIT** to upload the device package.

Now, you should see the following:

![Device Types](../img/devtype.png)

##Create the L4-L7 Device Manager
The F5 iWorkflow is a device manager used to manage the F5 Big-IP ADC. In the APIC GUI, click the following to configure the Device Manager:  
**L4-L7 Services -> Inventory -> Device Manager Type**  

Click the **ACTIONS** button at the Work pane and choose **Create Device Manager Type**

![Create Device Manager](../img/create-devmgr-1.png)

A new pop-up should appear to allow you to enter the device manager information. Enter the following information:

Vendor: **F5**  
Model: **iWorkflow**  
Version: **2.0-{TSTUDENT}**  
L4-L7 Service Device Type: **F5-iWorkflow-2.0-{TSTUDENT}**  
Device Manager: **Leave this field empty**

>It is extremely important to match the **Version** with the name of **L4-L7 Service Device Type** excluding the string - "**F5-iWorkflow-**".  

Click **SUBMIT** to accept the configuration.

![Enter Device Manager Info](../img/create-devmgr-2.png)

The Device Manager is now registered with the APIC and we need to associate this with your tenant. 

>**Switch to {TSTUDENT}-lab2 tenant when you reach this step** 

Navigate to your tenant to create a new L4-L7 Device Manager by clicking the following:  
**Tenants {TSTUDENT}-lab2 -> L4-L7 Services -> Device Managers**  

In the Work pane, click:
**ACTIONS -> Create Device Manager**  

![Create Tenant Device Manager](../img/tenant-devmgr-1.png)

A new pop-up should appear to allow you to Create  Device Manager in your tenant. Enter the following information:

Device Manager Name: **device-manager-{TSTUDENT}**  
Management EPG: **Leave this field empty since we use OOB to communicate**  
Device Manager Type: **F5-iWorkflow-2.0-{TSTUDENT}**  

Click the **+** to enter the device manager Management connectivity information:  
Host: **{TBIGIQIP}**  
Port: **443**  

Click **UPDATE** to accept.

Enter the Device Manager's login credential:  
Username: **admin**  
Password: **cisco123**  
Confirm Password: **cisco123**  

Click **SUBMIT** to accept the configuration.

![Tenant Device Manager Info](../img/tenant-devmgr-2.png)

Navigate to your tenant to confirm the Device Manager is created correctly:
**Tenants {TSTUDENT}-lab2 -> L4-L7 Services -> Device Manager -> device-manager-{TSTUDENT}**  

![Verify Device Manager](../img/tenant-verify-devmgr.png)

### --------------DELETE------------------

## ++++++ Payal suggestion ADD ++++++++
Fill out the variable file which represent's the application configuration that will be pushed to the Big-IP
- Give example of variable file and guide the user as to what to fill in 

```
onboarding: "yes"                                   Do you want to onboard the BIG-IP - Options: yes/no
banner_text: "--Standalone BIG-IP UnManaged ---"    SSH banner text
	
hostname: 'bigip.local'	                            Hostname of the BIG-IP (Part of onboarding)
	
ntp_servers:	                                    NTP servers to be configured (Part of onboarding)
 - '172.27.1.1'	
 - '172.27.1.2'	
	
dns_servers:	                                    DNS servers to be configured (Part of onboarding)
 - '8.8.8.8'	
 - '4.4.4.4'	
ip_version: 4	
	
module_provisioning:	                            Modules to be provisioned on BIG-IP (Part of onboarding)
 - name: 'ltm'	
   level: 'nominal'	
	
tenant_name_aci: "UM_F5_Tenant"	                    APIC tenant name
ldev_name_aci: "BIGIP_PHY"	                        APIC logical device cluster name
	
	
bigip_ip: 10.192.73.91	                            BIG-IP credentials
bigip_username: "admin"	
bigip_password: "admin"	
	
vlan_information:                                   VLAN to be added to BIG-IP
- name: "External_VLAN"                             VLAN’s match what is present in the logical device cluster BIGIP_PHY
  id: "1195"
  interface: "2.2"
- name: "Internal_VLAN"
  id: "1695"
  interface: "2.2"
	   
static_route:	                                    Add a static route
- name: "default"
  gw_address: "10.168.56.1"
  destination: "0.0.0.0"
  netmask: "0.0.0.0"
  
bigip_selfip_information:                           Self-IP to be added to BIG-IP, tag the appropriate VLAN to the respective Self-IP
- name: 'External-SelfIP'
  address: '10.168.68.10'
  netmask: '255.255.255.0'
  vlan: "{{vlan_information[0]['name']}}"
- name: 'Internal-SelfIP'
  address: '192.168.68.10'
  netmask: '255.255.255.0'
  vlan: "{{vlan_information[1]['name']}}"	
  
service: "yes"	                                    Do you want to configure HTTP service on the BIG-IP -Options: yes/no
	
vip_name: "http_vs"	                                VIP information (Part of configuring HTTP service)
vip_port: "80"	
vip_ip: "10.168.68.105"
	
snat: "None"                                        Options: ‘None/Automap/snat-pool name’
pool_name: "web-pool"	                            Pool Information (Part of configuring HTTP service)
pool_members:	
- port: "80"	
  host: "192.168.68.140"	
- port: "80"	
  host: "192.168.68.141"
```
### +++++++++ ADD +++++++++

##Create the L4-L7 Device for Service

Navigate to your tenant to create a new L4-L7 Device by clicking the following:  
**Tenants {TSTUDENT}-lab2 -> L4-L7 Services -> L4-L7 Devices**  

In the Work pane, click:  
**ACTIONS -> Create L4-L7 Devices**  
	
![L4-L7 device](../img/l4l7dev.png)

A new window should appear for you to create the L4-L7 Devices.

![LdevVIP](../img/ldevvip-lab2.png)

In the Create L4-L7 Devices window, enter the following:  
Managed: **CHECK**  
Name: **{TSNUM}-lab2-bigip**  
Service Type: **ADC**  
Device Type: **Virtual**  
VMM Domain, click the down arrow to select:  **CLBerlin2016**  
Mode: **Single Node**  
Device Package: **F5-iWorkflow-2.0–{TSTUDENT}**  
Model: **Unknown (Manual)**  
Context Aware: **Single**  
APIC to Device Management Connectivity: **Out-Of-Band**  
Username: **admin**  
Password: **cisco123**  
Confirm Password: **cisco123**  
	
![LdevVIP 1](../img/ldevvip-1-lab2.png)

In the _**Device 1**_, enter the following:  
Management IP Address: **{TBIGIPIP}**  
VM: Click the down arrow and select **CLBerlin2016/{TBIGIPVM}**  
Management Port: **https**  

Click the **+** to add a Device Interface:  
Name: **1_1**  
VNIC: **Network adapter 2**  

Click **UPDATE** to accept the Device Interface configuration.  

![CDev](../img/cdev.png)

Normally, we will accept the _**Cluster Management IP**_, which is the same as the Device 1’s management IP. However, since we are going to use the iWorkflow to manage BIG-IP, the _Cluster IP will be changed to the iWorkflow’s IP_. The device will eventually ignore this setting and it will use the Device Manager information configured earlier to establish communication .  
Management IP Address: **{TBIGIQIP}**  
Management Port: **https**  
Device Manager: **{TSTUDENT}-lab2/device-manager-{TSTUDENT}**  

Click the **+** to add the first Logical Interface:  
Type: **consumer**  
Name: **external**  
Concrete Interface: **Device1/1_1**  

Click **UPDATE** to accept the consumer interface configuration.  

![CDev 1](../img/cdev-1-lab2.png)

Click the **+** to add the second Logical Interface:  
Type: **provider**  
Name: **internal**  
Concrete Interface: **Device1/1_1**  

Click **UPDATE** to accept the consumer interface configuration.  

![CDev 2](../img/cdev-2-lab2.png)

Why am I associating both consumer and provider interface to the same concrete interface? We are doing this because we are using the ADC in one-arm mode. There is only one interface!  

Click **NEXT** to accept the **Step 1 > General**.  

We would like to set up some basic information on the Big-IP by choosing the **All Parameters** tab.  

![CDev Step 2](../img/ldevvip-step2.png)

Click **>** to expand the field **Device Host Configuration** and enter the following parameters and click **UPDATE** to save the change:
Host Name: **{TBIGIPVM}.lab.test.local**  
NTP Server: **172.21.208.254**

![CDev System Parameters](../img/ldevvip-step2-info.png)  

Click **FINISH** to accept the **Step 2 > Device Configuration**.

Navigate to the newly created L4-L7 Device to verify its Configuration State is stable:
	**Tenants {TSTUDENT}-lab2 -> L4-L7 Services -> L4-L7 Devices -> {TSNUM}-lab2-bigip**

In the Work pane, ensure the _Configuration State_ is _stable_, if the device is not stable, click the Faults tab and ensure no faults or all the faults are in clearing state.

![Stable device](../img/stable.png)

##Create L4-L7 Service Graph Templates

At this point, we are ready to create the L4-L7 Service Graph Template for the BIG-IP node.

To create a new Service Graph Template, click the following in the navigation pane:  
**Tenants {TSTUDENT}-lab2 -> L4-L7 Services -> L4-L7 Service Graph Template**  

In the Work pane:  
**ACTIONS -> Create L4-L7 Service Graph Template**  

In the new window, enter the following:  
Graph Name: **{TSNUM}-lab2-sg2**  
Graph Type: **Create a New One (should be the default)**  

Now, drag the Device Clusters to the right side of the window into the graph. You should be able to place the Node **{TSTUDENT}-lab2/{TSTUDENT}-bigip** between the **Consumer** EPG and the **Provider** EPG.  

Double click the word **N1** under the Node to change the name to **ADC**.  

Under **{TSTUDENT}-bigip** Information, click the **One-Arm** option for this graph.  

Select the Profile: **F5-iWorkflow-2.0–{TSTUDENT}/{TSTUDENT}-http-no-persistence**  

**Route Redirect**: leave it **UNCHECK** for now  

![Service Graph template](../img/template-lab2.png)

Click **SUBMIT** to continue.  

The new ADC L4-L7 Service Graph Template is now created and we are ready to deploy the BIG-IP with the pre-created **epg-l3out**  and  **web-epg** EPG.  

Let’s go back to the L4-L7 Service Graph Template and edit the Connections setting by clicking the following in the Navigation pane of your tenant:  
**Tenant {TSTUDENT} -> L4-L7 Services -> L4-L7 Service Graph Template  {TSNUM}-lab2-sg2**  
	
![Service Graph template 1](../img/template-1-lab2.png)

You can modify the Connections **C1** and **C2** to L3 Under Properties. C1 is the connector between the services device facing toward to the provider EPG, and C2 is the connector between the services device facing toward to the consumer EPG. In this case, the connection facing toward to a BD with subnet configured. Additionally, in ADC one-arm mode, there is only one connector facing both consumer and provider and we have to make the **Adjacency Type** the same.

C1 Adjacency Type: **L3**  
Click **UPDATE**  
C2 Adjacency Type: **L3**  
Click **UPDATE**  
Click **SUBMIT** to accept the change  
	
![Connectors](../img/templateconn.png)

##Deploy the Service Graph Part 1

To deploy the service graph, click the following in the Navigation pane of your tenant:  
**Tenants {TSTUDENT}-lab2 -> L4-L7 Services -> L4-L7 Service Graph Template**

Select the Service Graph Template you just created from the Work pane. Right click and choose the option to **Apply L4-L7 Service Graph Template**.  

![Apply Service Graph template](../img/apply-lab2.png)

In the new window, you will have the ability to choose which EPGs the Service Graph will be inserted between. Select the following for the EPG information:  
Consumer EPG / External Network: **{TSTUDENT}-lab2/{TSTUDENT}-lab2-l3out/epg-l3out**  
Provider EPG / External Network: **{TSTUDENT}-lab2/app/epg-web-epg**  

Under Contract Information, use the option to create a new Contract:  
Create a New Contract: **SELECTED**  
Contract Name: **{TSNUM}-lab2-cntr2**  
No Filter (Allow All Traffic): **CHECKED**  

![Apply Service Graph template 1](../img/apply-1-lab2.png)

Click **NEXT** to continue to the next screen.

##Deploy the Service Graph Part 2

A new window to apply the service graph template will now appear. This window will show the Service Graph Template that you created earlier. 

In addition to the Service Graph Template, there are some options that need to be selected to deploy the BIG-IP with a Service Graph. As illustrated in the beginning of this lab, the ADC will now be placed in its own Bridge Domain along with the load balancing VIP. Under the **{TSTUDENT}-lab2/{TSNUM}-lab2-sg2** information, you need to choose the appropriate connector information:

Under the Connector, choose the following:  
Type: **General**  
BD: **{TSTUDENT}-lab2/vip-bd**  
Cluster Interface: **internal**  

Click **NEXT** to continue to the next screen.

![Apply Service Graph template 2](../img/apply-2-lab2.png)

##Deploy the Service Graph Part 3

A new window for the BIG-IP parameters will now appear. In this window, you will have the ability to modify the parameters to be deployed to the BIG-IP. Let’s modify some parameters to push the Service Graph into the BIG-IP.

Under Feature, All should be selected And the Parameters should be All Parameters.

![Parameters 1](../img/params-1.png)

Once you click the All Parameters tab, the folder and parameters will appear. To edit the parameter, you need to expand the parameter by clicking the **>** and double click the field to change the parameter’s name and value. Let’s edit the following parameters:  
Under **Device Config**  
Press **>** to expand the **Network** configuration folder  
Press **>** to expand the folder **InternalSelfIP**   
Double click the parameter **Enable Floating?** and select **No** as the value  
Click **UPDATE** to apply  
Double click the parameter **Internal Self IP Address** and enter **{TL2F5INTSIP}** as the value  
Click **UPDATE** to apply  
Double click the parameter **Internal Self IP Netmask** and enter **255.255.255.0** as the value  
Click **UPDATE** to apply  
Double click the parameter **Port Lockdown** and select **Default** as the value  
Click **UPDATE** to apply  

Press **>** to expand the folder **Route**  
Double click the parameter **Destination IP Address** and enter **0.0.0.0** as the value  
Click **UPDATE** to apply  
Double click the parameter **Destination Netmask** and enter **0.0.0.0** as the value  
Click **UPDATE** to apply  
Double click the parameter **Next Hop Router IP Address** and enter **{TL2F5VIPGW}** as the value  
Click **UPDATE** to apply  

Under **Function Config**  
Press **>** to expand the **NetworkRelation** configuration folder  
Double click the parameter **Select Network** and select **Network** as the network relation  
Click **UPDATE** to apply  

Under **Function Config**  
	Press **>** to expand the **{TSTUDENT}-http-no-persistence-Default** configuration folder  
	Double click on the name and delete **Default**  
Click **UPDATE** to apply  
	Press **>** to expand the **Pool Members** folder  
	Press **>** to expand the **Member** folder  
Double click to enter value into the **IPAddress** field: **{TVM2IP}**  
Click **UPDATE** to apply  
Double click the parameter **Port** to accept the TCP port: **80**  
Click **UPDATE** to apply  

Press the **+** on the left side next to the **Member** folder to add a new member  
Click **UPDATE** to accept the name  

Press **>** to expand the **Member** folder (the name will be populated as **member2**)  
Double click to enter value into the **IPAddress** field: **{TVM3IP}**  
Click **UPDATE** to apply  
Double click the parameter **Port** and enter the TCP port: **80**  
Click **UPDATE** to apply  

Back to the **{TSTUDENT}-http-no-presist** configuration folder  
Double click to enter value into the **Address** field (Name **pool_addr**): **{TL2F5VIP}**  
Click **UPDATE** to apply  
Double click the parameter **Port** to accept the TCP port: **80**  
Click **UPDATE** to apply  
Double click to enter value into the **SNATConfig** field: **automap**

Click **FINISH** to finish the configuration of the BIG-IP parameters  

![Parameters 2](../img/params-2.png)  
![Parameters 3](../img/params-3.png)

##Verifying the BIG-IP Service Graph Deployment on APIC

You can now verify if APIC has deployed the service graph correctly. First, navigate the following:  
**Tenant {TSTUDENT}-lab2 -> L4-L7 Services -> Deployed Devices**  

You should be able to see a screen similar to the following. The State should say **_allocated_**.  

![Deployed device](../img/deployed-lab2.png)

Next, you can navigate to:  
**Tenant {TSTUDENT}-lab1 -> L4-L7 Services -> Deployed Graph Instances**  

The Deployed Graph Instances state should say **_applied_**.  

![Deployed graph](../img/deployedgraph.png)

##Verifying the BIG-IP Service Graph Deployment

Log into the F5 iWorkflow **{TBIGIQIP}** with the following username and password from the web browser (if the previous session has timed out):  
iWorkflow: **[https://{TBIGIQIP}](https://{TBIGIQIP})**  
Username: **admin**  
Password: **cisco123**  

Under the iWorkflow **_Clouds and Services_** configuration. In the Work pane, under **Services** and **Nodes**, you should see the deployed application, real servers and APIC tenant name listed similar to the following:

![IQ Verification](../img/iq-ver.png)

Double click on the **_Services_** and you should see the screen similar to the following:

![IQ Apps](../img/iqapp.png)

>Note - the Services name will contain the deployed partition number in BIG-IP is the same as the APIC virtual device number. 

You can also see that the Status of the services says **_Application Service is healthy_**. The Application Type is the Catalog template we have created earlier, **{TSTUDENT}-http-no-persistence** with the cloud connector as your student number, **{TSTUDENT}**. 

Under the Application Template, you will see the Pool Addr in the notation of the VIP plus the Route-Domain (VRF): **{TL2F5VIP} 80**

The Pool Members are the real servers’ IP, **{TVM2IP}** and **{TVM3IP}** and listening port, in our case, it is port **80**.

Now, let’s log into the F5 BIG-IP **{TBIGIPIP}** with the following username and password from the web browser (if the previous session has timed out):  
BIG-IP: **[https://{TBIGIPIP}](https://{TBIGIPIP})**  
Username: **admin**  
Password: **cisco123**  

On the **Main** menu, click **Local Traffic -> Network Map**. Then on the top right corner, next to the **_Log out_** button, click the drop down to select the newly created **Partition** (please note that this reflects the APIC Virtual Device ID):  

![IP Partition](../img/partition.png)

Once you are in the partition, click **Local Traffic -> Network Map**. You should be able to see the virtual server is in green along with its pool and pool members.

![Network Map](../img/netmap.png)

On the left Navigation menu, click the **Local Traffic -> Virtual Servers** and you should be able to see the brief Virtual IP information. You can see that the VIP is currently listening on HTTP port 80.

The number (in this example, **_2854_** ) after the **_%_** mark represents the route domain (RD) number.  There will be a RD number assign to each APIC partition, which equivalent to an ACI L3 VRF.  This allows BIG-IP to provide multi-tenancy support in ACI environment.

![VIP](../img/vip.png)

In the **Virtual Server List**, click the **Name** in the hyperlink and you will see the **Property** of the Virtual Server with more detailed information. The configured the parameters will appear here. You should also notice the Source Address Translation option is set to **Auto Map**.

![VIP 1](../img/vip-1.png)

Click the **Resources** tab and you should see the both **Default** and **Fallback** persistence profiles are set to **None**.  

>Remember that we did this in iWorkflow?

![VIP 2](../img/vip-2.png)

Click **Local Traffic -> Pools** and you should see the brief information of the real server pool information:

![Pool](../img/pool.png)

Click the hyperlink under **Name** and you should be directed to the Pool **Properties** page. Now click the **Members** tab and you should see the real servers (pool members) we configured when we were deploying the service graph.

![Pool Member](../img/pool-member.png)

Let’s go back to the Navigation pane and click the **iApps -> Application Services**, you should see the **Application Services List** and the iApp template we were using earlier.

![iApps](../img/iapps.png)

Click the hyperlink in **Name** and you will be directed to the Application Services **Components**. By using iApps template, you can configure a full-features virtual server by specifying customized parameters exposed to APIC.

![Components](../img/components.png)

Go to your RDP command line and ping **{TL2F5VIP}** (VIP). This should succeed.

You can now verify the Virtual Server (or VIP) by using your browser and entering the VIP into the address window:  

URL: **[http://{TL2F5VIP}](http://{TL2F5VIP})**  

You should see the page with the hostname of your VMs similar to the following:   
	
![Page 1](../img/page1.png)

If you press the enter button (do not use the refresh button of your browser) at the IP address **{TL2F5VIP}** at the web browser, you should see a different page. You can repeat this multiple time to see the page load balanced between different web servers.

![Page 2](../img/page2.png)

Now, we have verified connectivity to the web server via the ADC VIP, let's inspect the web server and see which client IP is in the access log.

Use the Putty SSH client in your RDP and connect to the first web server VM and verify its access log. For example, you can SSH into VM, {TSTUDENT}-vm02, via its L2 only network connection by using the following:

IP address: **{TVM2L2}**  
Username: **root**  
Password: **ciscolive.2017**  

After logged in, issue the following command:  

```
tail -f /var/log/httpd/access_log
```

You should see an output similar to the following:  

```
69.2.135.10 - - [06/Feb/2017:20:11:52 -0500] "GET /" 200 770 "-" "-"
69.2.135.10 - - [06/Feb/2017:20:11:56 -0500] "GET /" 200 770 "-" "-"
```

What you see above is the HTTP health probe from the Big-IP. Let's use the web browser in your RDP session to browse the web server via the ADC VIP for couple times. You should see the browser switch between VM2 and VM3.

Now, let's inspect the http access log and you will notice an output similar to the following:  

```
69.2.135.10 - - [06/Feb/2017:20:14:05 -0500] "GET / HTTP/1.1" 200 770 "-" "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.87 Safari/537.36"

```

**Why we are seeing the web server accessing by the Big-IP instead of the web browser in the RDP server?**  

It is because we are using the BIG-IP feature **Auto Map** to Source NAT the client's source IP.  

**What happens if we want to see the real IP of the client?**  

##Reconfiguring the BIG-IP Service Graph Deployment with Policy-Based Redirect

We can accomplish this by using the Policy-Based Redirect feature in the ACI fabric starting version 2.0.

Let's reconfigure the ACI fabric and the ADC to see how Policy-Based Redirect in action.

First, we will undeploy the service graph. In the APIC GUI, click the following:   
 
**Tenants {TSTUDENT}-lab2 -> Security Policies -> Contracts -> {TSNUM}-lab2-cntr2 -> Subjects**  

In the Work pane, undeploy the service graph by clicking the **x** in the **Service Graph** field  

![Undeploy SG](../img/pbr-undeploy.png)  

Click **Submit** to accept the change  

Navigate to **Tenants {TSTUDENT}-lab2 -> L4-L7 Services -> Deployed Devices**, you might see the deployed devices state as deletepending. The **deletepending** state means the device package is trying to unconfigure the service graph from the ADC, it will wait for the next audit state to confirm the deletion.  

![Delete Pending](../img/pbr-deletepending.png)

Once the service graph is undeployed successfully, the Work pane under **Deployed Devices** will be emptied.  

In order to configure policy-based redirect, we need to disable the **Endpoint Dataplane Learning** under the **vip-bd**. To preserve the client's real IP, the client's real IP will remain the same after the packet is load balanced by the ADC. If dataplane learning is enabled, the COOP database will program the leaf where the ADC is connected to with the client's IP and this will cause communication issue between the client and other EPs in the ACI fabric. 

In the APIC GUI, click the following:  
**Tenants {TSTUDENT}-lab2 -> Networking -> Bridge Domains -> vip-bd**  
Uncheck **Endpoint Dataplane Learning**  

![Disable EP Learning](../img/pbr-eplearning.png)  

Click **Submit** to accept the change.  

Next, we need to configure the policy-based redirect in the ACI fabric. First, we need to obtain the interface MAC address of the BIG-IP using the BIG-IP UI using:

BIG-IP: **[https://{TBIGIPIP}](https://{TBIGIPIP})**  
Username: **admin**  
Password: **cisco123**  

Click **Network -> Interfaces** and copy down the interface **1.1**'s **MAC Address**

![BIGIP MAC](../img/pbr-bigipmac.png)  

For example, the above illustration has the MAC address as **0:50:56:9c:ab:cd**. When we are configuring the ACI Policy-Based Redirect, we need to enter the MAC address with an appending zero, such as **00:50:56:9c:ab:cd**.

>Please use the MAC Address from the BIG-IP UI. Do **NOT** copy the mac address from this guide.

In the APIC GUI, to configure policy-based redirect policy, navigate to the following:

**Tenants {TSTUDENT}-lab2 -> Networking -> Protocol Policies -> L4-L7 Policy Based Redirect**  

In the Work pane, click **Actions**

![PBR1](../img/pbr1.png)  

In the window appear in the Work pane, enter the following:

Name: **{TSTUDENT}-pbr**  
IP: **{TL2F5INTSIP}**  
MAC: **MAC-Address-You've-Recorded-Earlier-Starts-With-00**  
Click **Update** to accept the changes  

![Config PBR](../img/pbr-config.png)

Click **Submit** to accept the configuration

We need to also update the service graph template to enable Policy-Based Redirect. First, we need to enable service graph **Direct Connect** in the L4-L7 Service Graph Template. In the APIC GUI, navigate to the following:

**Tenants {TSTUDENT}-lab2 -> L4-L7 Services -> L4-L7 Service Graph Templates -> {TSNUM}-lab2-sg2**  
Select **Policy**.  
Scroll down to both **C1**, double click and select **Direct Connect** as **True**.  
Repeat the same step for **C2** to set **Direct Connect** as **True**:  

![PBR Direct Connect](../img/pbr-directconnect.png)

Second, we need to enable the Policy-Based Redirect function in the service graph template. In the APIC GUI, navigate to the following:  

**Tenants {TSTUDENT}-lab2 -> L4-L7 Services -> L4-L7 Service Graph Templates -> {TSNUM}-lab2-sg2 -> Function Node - ADC**  

In the Work pane, configure the **Route Redirect** as **Redirect**  

![Function Node PBR](../img/pbr-funcnode.png)  

Click **Submit** to accept the configuration

Next, we need to update the L4-L7 Devices Selection Policy to apply the **Policy Based Redirect** to the service graph Logical Interface Context - provider. In the APIC GUI, navigate to the following:

**Tenants {TSTUDENT}-lab2 -> L4-L7 Services -> Devices Selection Policies -> {TSNUM}-lab2-cntr2-{TSNUM}-lab2-sg2-ADC -> provider**  

In the Work pane, L4-L7 Policy-Based Redirect, select the option **{TSTUDENT}-lab2/{TSTUDENT}-pbr**  

![PBR LDevCtx](../img/pbr-ldevctx.png)

Click **Submit** to accept the configuration

Before we can instantiate the service graph again, we need to change the SNAT configuration to preserve client IP address. In the APIC GUI, navigate to the following:

**Tenants {TSTUDENT}-lab2 -> Applicaiton Profiles -> app -> Application EPGs -> EPG web-epg -> L4-L7 Service Parameters**  

In the Work pane, expand the service parameters and look for the folder **{TSTUDENT}-http-no-persistance**. Expand the folders until you find the parameter called **vs__SNATConfig**:  
Double click the parameter value and enter **None**.  
Click **UPDATE** to accept the changes.

>**Please click the UPDATE button. Do not just press the ENTER key**

![PBR L4L7 Param](../img/pbr-l4l7param.png)

The service graph is ready for Policy-Based redirect. We can instantiate the service graph by reapply the service graph template to the contract. In the APIC GUI, navigate to the following:

**Tenants {TSTUDENT}-lab2 -> Security Policies -> Contracts -> {TSNUM}-lab2-cntr2 -> Subjects**  

Select the service graph template **{TSTUDENT}-lab2/{TSNUM}-lab2-sg2**

![PBR Reapply Cntr](../img/pbr-reapplycntr.png)

Click **Submit** to accept the configuration

##Verifying the BIG-IP Service Graph Deployment with Policy-Based Redirect on APIC

You can now verify if APIC has deployed the service graph with policy-based redirect correctly. Navigate to the following:  
**Tenant {TSTUDENT}-lab2 -> L4-L7 Services -> Deployed Devices**  

You should be able to see a screen similar to the following. The State should say **_allocated_**.  

![Deployed device](../img/deployed-lab2.png)

Next, you can navigate:  
**Tenant {TSTUDENT}-lab1 -> L4-L7 Services -> Deployed Graph Instances**  

The Deployed Graph Instances state should say **_applied_**.  

![Deployed graph](../img/deployedgraph.png)  

Log into the F5 iWorkflow **{TBIGIQIP}** with the following username and password from the web browser (if the previous session has timed out):  
iWorkflow: **[https://{TBIGIQIP}](https://{TBIGIQIP})**  
Username: **admin**  
Password: **cisco123**  

Under the iWorkflow **_Clouds and Services_** configuration. In the Work pane, under **Services** and **Nodes**, you should see the deployed application, real servers and APIC tenant name listed similar to the following:

![IQ Verification](../img/iq-ver.png)

Double click on the **_Services_** and you should see the screen similar to the following:

![IQ Apps](../img/iqapp-2.png)

>Note - the Services name will contain the deployed partition number in BIG-IP which is the same as the APIC virtual device number. **This number can be different from the previous virtual device number when we were deploying the service graph using Auto-Map** 

You can also see the Status of the services says **_Application Service is healthy_**.  

Under the Application Template, you will see the Pool Addr in the notation of the VIP plus the Route-Domain (VRF): **{TL2F5VIP} 80**

The Pool Members are the real servers’ IP, **{TVM2IP}** and **{TVM3IP}** and the listening port, in our case, it is port **80**.

Now, let’s log into the F5 BIG-IP **{TBIGIPIP}** with the following username and password from the web browser (if the previous session has timed out):  
BIG-IP: **[https://{TBIGIPIP}](https://{TBIGIPIP})**  
Username: **admin**  
Password: **cisco123**  

On the **Main** menu, click **Local Traffic -> Network Map**. Then on the top right corner, next to the **_Log out_** button, click the drop down to select the newly created **Partition**:  

![IP Partition](../img/partition.png)

Once you are in the partition, click **Local Traffic -> Network Map**. You should be able to see the virtual server is in green along with its pool and pool members.

![Network Map](../img/netmap.png)

On the right Navigation menu, click the **Local Traffic -> Virtual Servers** and you should be able to see the brief Virtual IP information. You can see that the VIP is currently listening on HTTP port 80.

![VIP](../img/vip.png)

In the **Virtual Server List**, click the **Name** in the hyperlink and you will see the **Property** of the Virtual Server with more detailed information. Scroll down and you should notice the Source Address Translation option is now set to **None**.

![VIP 1](../img/pbr-vip-1.png)

Click **Local Traffic -> Pools** and you should see the brief information of the real server pool information:

![Pool](../img/pool.png)

Click the hyperlink under **Name** and you should be directed to the Pool **Properties** page. Now click the **Members** tab and you should see the real servers (pool members).

![Pool Member](../img/pool-member.png)

Let’s go back to the Navigation pane and click the **iApps -> Application Services**, you should see the **Application Services List**  and the iApp template.

![iApps](../img/iapps.png)

Click the hyperlink in **Name** and you will be directed to the Application Services **Components**. 

![Components](../img/components.png)

Go to your RDP command line and ping **{TL2F5VIP}** (VIP). This should succeed.

You can now verify the Virtual Server (or VIP) by using your browser and entering the VIP into the address window:  

URL: **[http://{TL2F5VIP}](http://{TL2F5VIP})**  

You should see the page with the hostname of your VMs similar to the following:   
	
![Page 1](../img/page1.png)

Press the enter button (do not use the refresh button of your browser) at the IP address **{TL2F5VIP}** at the web browser, you should see a different page and the VIP is load balanced by the ADC.

![Page 2](../img/page2.png)

Now that we have verified connectivity to the web server via the ADC VIP, let's inspect the web server and see which client IP is in the access log.

Use the Putty SSH client in your RDP and connect to the first web server VM and verify its access log entries. For example, you can SSH into VM, {TSTUDENT}-vm02, via its L2 only network connection by using the following:

IP address: **{TVM2L2}**  
Username: **root**  
Password: **ciscolive.2017**  

After logged in, issue the following command:  

```
tail -f /var/log/httpd/access_log
```

You should see an output similar to the following:  

```
69.2.135.10 - - [06/Feb/2017:20:11:52 -0500] "GET /" 200 770 "-" "-"
69.2.135.10 - - [06/Feb/2017:20:11:56 -0500] "GET /" 200 770 "-" "-"
```

This is the HTTP health probe from the Big-IP. Let's use the web browser in your RDP session to browse the web server via the ADC VIP for multiple times. You should see the browser switch between VM2 and VM3.

Now, let's inspect the HTTP access log and you will notice an output similar to the following:  

```
199.253.253.12 - - [07/Feb/2017:22:29:07 -0500] "GET / HTTP/1.1" 200 770 "-" "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.87 Safari/537.36"
```  

The IP address in the HTTP access log is no longer showing the the interface IP of the BIG-IP and it contains the client's IP in the HTTP request instead. The ACI fabric uses the Policy-Based Redirect to return the response from the web server to the client. This is similar to the traditional network concept of Policy-Based Routing. 

>**Congratulations! This session of the lab is completed, please proceed to the Bonus Lab session via the menu**
