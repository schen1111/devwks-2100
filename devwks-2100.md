# DevNet Workshop - DEVWKS-2100

# Build Ansible Plays to Automate Crosswork API Invocations


### Sanka Chen, CCIE #49686, Technical Leader, Cisco Systems

### Weigang Huang, CCIE #49667, Customer Delivery Software Architect, Cisco Systems


## Workshop guide

### Use Case Summary
This workshop provides walkthrough for few use cases to automate Cisco Crosswork API by using Ansible. Participants can leverage the provided use cases to start their Crosswork API automation journey.

#### Use-case #1: Device onboarding
#### Use-case #2: Attach devices to CDG (Crosswork Data Collector)
#### Use-case #3: Change the admin state (up or down) of the device in Crosswork

<br/><br/>
### API Authentication (Common Across all Use Cases)
In order to invoke any Crosswork API, you must first obtain a JWT (JSON Web Token). The JWT needs to be in every HTTP header of API calls using the Bearer Authorization. This is a two step process. 

1. Request a TGT (Ticket Granting Ticket) using your Crosswork GUI credential 
2. Request a JWT using the TGT in step 1

#### Task 1: Perform Authentication using Postman
1. Open Postman (shortcut in desktop)
2. Expand this collection "DEVWKS-2100"
3. Expand CW Authentication folder. You should see two post requests
4. Select "cw-api-get-ticket-step1" request and hit send. You should see a TGT in the response
5. Select "cw-api-get-jwt-step2" request and hit send. You should see a JWT in the response

#### Task 2: Perform Authentication using Ansible
1. Inspect this playbook: ```ansible_project/roles/get_crosswork_authentication/tasks/main.yml```([content link](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/roles/get_crosswork_authentication/tasks/main.yml)). This playbook minmics the two steps in Postman. 
	* The variables used in this playbook are referenced in ```ansible_project/global_variable.yml```([content link](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/global_variable.yml))
	* The output of each steps is saved to a variable within Ansible
	* Please use Ansible-vault to store the password variable in production scenario
2. Open Ubuntu (shortcut in desktop)
3. Go to directory: 
```
cd /mnt/c/Users/Administrator/Downloads/devwks-2100/ansible_project
```

4. Execute this playbook ([content link](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/get_crosswork_authentication_play.yml)):
```
ansible-playbook get_crosswork_authentication_play.yml
```
5. You should see the the TGT and JWT in the debug output from the Ansible play


<br/><br/>
### Use-case #1: Device Onboarding
This usecase demonstrates how to leverage Ansible to onboard devices to Crosswork.

#### Task 3: Understand Device Onboarding
1. Most of you will probably get a list of devices to onboard via an Excel sheet. One of the easier ways for Ansible to consume the data is to convert the Excel into Yaml file. You can do it fairly easily using online tools
2. (Not required for this lab) - Covert Excel to CSV
3. (Not required for this lab) - Covert CSV to YAML by using online converter. Converter can be found via search engines
4. (Not required for this lab) - Use online YAML beautifier to properly format the converted YAML file. YAML beautifierc can be found via search engines.
5. Inspect ```ansible_project/global_variable.yml``` ([content link](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/global_variable.yml)) to see the converted YAML under the devices section
6. Inpsect the Ansible tasks ```/ansible_project/roles/device_onboarding/tasks/main.yml```([link content](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/roles/device_onboarding/tasks/main.yml)) to onboard the devices. The task will generate API payload with each device's information from the global_variable.yml using the Jinja2 template
	* The task will loop through the devices variable ([content link](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/global_variable.yml))
	* For each item in the devices variable list, it will generate an API payload by using the Ansible template module
	* The template module mainly consist of two parts:
		* ```src``` - source of template 	
		* ```dest``` - destionation location to render the template output
	* Inspect this Jinja2 template ```ansible_project/roles/device_onboarding/tasks/main.yml```([content link](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/roles/device_onboarding/templates/add-device.j2))
		* All the variables referenced in the template can be found in the devices variable ```ansible_project/global_variable.yml``` ([content link](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/global_variable.yml))
	* By using the template module, you can easily generate large amount of API payloads

```yaml
- name: create API payloads for adding new devices to CNC
  template:
    src: "../templates/add-device.j2"
    dest: "{{playbook_dir}}/temp_folder/device_onboarding_api_payload/add-device-{{item.host_name}}.json"
  loop: "{{devices}}"
  loop_control: 
    label: "{{item.host_name}}"
```

#### Task 4: Perform Device Onboarding
1. Go to directory: ```cd /mnt/c/Users/Administrator/Downloads/devwks-2100/ansible_project```
2. Execute the Ansible playbook to prepare for this task:```ansible-playbook cisco_live_prepare_for_device_onboarding_play.yml```
3. Log into Crosswork Web GUI in the Windows VM: ```https://198.18.134.219:30603/#/inventory/overview```. You can view devices that are currently onboarded to Crosswork. There should be none at this point
	* Username: ```admin```
	* Password: ```C!sco12345```
4. Execute this playbook to onboard devices: This will onboard all devices listed in the ```ansible_project/global_variable.yml``` ([content link](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/global_variable.yml))

```
ansible-playbook device-onboarding-play.yml
```
Contents of ```ansible-playbook device-onboarding-play.yml``` ([content link](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/device-onboarding-play.yml)). 
5. Check the Crosswork Web GUI in the Windows VM for the newly onboarded devices: ```https://198.18.134.219:30603/#/inventory/overview```



