Ansible, F5 and Cisco ACI dCloud Lab Guide
==========================================

This lab guide will cover on how to use Ansible to automate a F5 BIG-IP and Cisco ACI environment

Goal using ansible is to achieve the following tasks:
- Perform L2-L3 stitching between the Cisco ACI fabric and F5 BIG-IP
- Deploy an application on BIG-IP
- Automate elastic workload commision/decommission

We will be using Ansible Tower to execute all the tasks. 

Ansible tower credentials:

- https://198.18.134.151 (admin/C1sco12345) - a browser shortcut is also present with the name 'Ansible AWX'

|

|
	
.. toctree::
   :caption: Autommate F5 and Cisco ACI using Ansible
   :glob:
   :maxdepth: 2
   :hidden:

   ansible_tower_walkthrough.rst
   dynamic_ep.rst
   