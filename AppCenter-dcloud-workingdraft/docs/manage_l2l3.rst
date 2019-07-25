Manage L2-L3 configuration
==========================

Now that we have performed some Day0 configuration for the network and also the application. 

Lets look at how we can managing DayN configuration using the application.

Re-deploy service graph
```````````````````````
Assume someone from your team also has access to the APIC and without understanding the impact re-deploys the service graph which has already been stitched to the BIG-IP.

Let's take a look at how the application and help rectify the issue.

First let's redeploy the graph. Login to the APIC and navigate to Tenant LAX->Contracts->Standard->BIGIP-VE-Standalone-Contract->Subject

Scroll to the bottom and find the L4-L7 Service Graph parameter

.. image:: ./_static/managel2l3-1.png

..

Clear that field (once you hover over it you will see a X sign, click on that). The field will get cleared. Then click submit and click on Submit Changes

Check under L4-L7->Services->Deployed Graph Instance  - there is no deployed graph now

Let's re-deploy it, back to the contract and subject menu. Let's assign the 2ARM-Template back to the L4-L7 Service Graph paramter

Check the deployed graph instance and also the VLANs allocated under the function node. They will be different from what was assigned earlier

.. image:: ./_static/managel2l3-2.png

..

Now let's go back to the F5 ACI ServiceCenter , lets see the VLAN table, we dont see any infomration. This is because now the VLANS on the APIC has changed and there is no way for the application to co-relate the VLANS 1001 and 1002 anymore (since those VLANS dont exist anymore on the APIC)

Now go to the L2-L3 Network management table

Move the new VLANs from Available to Selected. 

.. image:: ./_static/managel2l3-3.png

..

Scroll to the bottom, you will see a warning sign next to each of the VLAN's. Hover over it to view what it says.

.. image:: ./_static/managel2l3-4.png

..

Next click on Manage selected, Click continue

Change the VLAN to reflect the new VLAN in the form and click submit. In my case
- 1002 changed to 1171
- 1001 changed to 1003

After you submit you will see the configuration is consistent again and no warning sign anymore

.. image:: ./_static/managel2l3-5.png

..

The application has helped you see the configuration discrepency and provided you will an interface to fix that discrepency without logging directly into the BIG-IP

You can at this point also login to the BIG-IP and view the VLAN change

Modify BIG-IP configuration out-of-band
```````````````````````````````````````

- Change vlan ID BIG-IP
- Go to visibility - only one why?
- Let's sync App DB back to BIG-IP - all good
- Do the process again, change vlan id on BIG-IP 
  - Sync to BIG-IP to App DB , what happens and why?