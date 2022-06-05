## DevNet Workshop - DEVWKS-2100

## Build Ansible Plays to Automate Crosswork API Invocations


### Sanka Chen, CCIE #49686, Technical Leader, Cisco Systems

### Weigang Huang, CCIE #49667, Customer Delivery Software Architect, Cisco Systems


## Workshop guide

### Use Case Summary
This workshop provides walkthrough for few use cases to automate Cisco Crosswork API by using Ansible. Participants can leverage the provided use cases to start their Crosswork API automation journey.

#### Use-case #1: Device Onboarding
#### Use-case #2: Attach Devices to CDG (Crosswork Data Collector)
#### Use-case #3: Change the Admin State (up or down) of the Device in Crosswork

<br/><br/>
### Get the Latest Ansible Code from Github
#### Task 1: Get the Latest Ansible Code from Github
1. Open Ubuntu (shortcut in desktop)
2. Go to directory: 
```
cd /mnt/c/Users/Administrator/Downloads/devwks-2100/ansible_project
```
3. Perform git pull:
```
git pull
```

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
This use-case demonstrates how to leverage Ansible to onboard devices to Crosswork.

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
		* ```src``` - Jinja2 template source path
		* ```dest``` - destionation location to render the template output
	* Inspect this Jinja2 template ```ansible_project/roles/device_onboarding/templates/add-device.j2```([content link](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/roles/device_onboarding/templates/add-device.j2))
		* All the variables referenced in the template can be found in the devices variable ```ansible_project/global_variable.yml``` ([content link](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/global_variable.yml))
	* Example of the payloads```ansible_project/temp_folder/device_onboarding_api_payload```: [content link](https://github.com/schen1111/devwks-2100/tree/main/ansible_project/temp_folder/device_onboarding_api_payload)
	* Note that the API payload files are generated for reference only so you can see the payloads. The task that will invoke the onbarding API will able to generate the payload on the fly without writing the payload to disk.
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
1. Go to directory: 
```
cd /mnt/c/Users/Administrator/Downloads/devwks-2100/ansible_project
```
2. Execute the Ansible playbook to prepare for this task:
```
ansible-playbook cisco_live_prepare_for_device_onboarding_play.yml
```
3. Log into Crosswork Web GUI in the Windows VM: ```https://198.18.134.219:30603/#/inventory/overview```. You can view devices that are currently onboarded to Crosswork. There should be none at this point
	* Username: ```admin```
	* Password: ```C!sco12345```
4. The devices listed in ```ansible_project/global_variable.yml```([content link](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/global_variable.yml)) will be onboarded to Crosswork
5. Execute the below playbook to onboard devices ([content link](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/device-onboarding-play.yml)):
```
ansible-playbook device-onboarding-play.yml
```
5. Check the Crosswork Web GUI in the Windows VM for the newly onboarded devices: ```https://198.18.134.219:30603/#/inventory/overview```


<br/><br/>
### Use-case #2: Attach Devices to CDG (Crosswork Data Collector)
This use-case demonstrates how to leverage Ansible to attach onboarded devices to CDG (Crosswork Data Gateway)

#### Task 5: Understand Attach Devices to CDG
1. Once devices are onboarded to Crosswork, they must be attached to CDG before becoming operational
2. Crosswork can have multiple CDGs for redundancy and scalability reasons. In this lab, there will be only one CDG. If there are mutiple CDGs, devices can be attached to different CDG
3. You can define which device is attached to which CDG by changing the vdguuid for each device under ```ansible_project/global_variable.yml``` ([content link](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/global_variable.yml))
4. Inspect the Ansible tasks: ```ansible_project/roles/attach_device_to_cdg/tasks/main.yml```([content link](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/roles/attach_device_to_cdg/tasks/main.yml))
	* Task```invoke nodes query API to obtain device UUID for devices that are onboarded to CW``` will make an API call to obtain devices info which includes the UUID for each device.
	* Task```create a new dictionary with Deivce IP and Device vdguuid``` will create a new dictionary variable which device IP as the key and the vdguuid as the value. The next task will leverage this variable to look up which device should be attach to which CDG.
	* Task ```create a list of dictionaries for the Jinja2 template``` will create a list of dictionaries with hostname, deivce ip and device uuid and planned vdguuid for devices that aren't attached to a CDG
5. Task```create API payloads for attach CDG to devices```will generate attach device to CDG API payload from the variable ```device_uuid_cdguuid_list``` using the Jinja2 template

	```yaml
	- name: create API payloads for attach CDG to devices
	  template:
	    src: "../templates/attach-cdg.j2"
	    dest: "{{playbook_dir}}/temp_folder/attach_device_to_cdg_api_payload/attach-cdg-{{item.hostname}}.json"
	  loop: "{{device_uuid_cdguuid_list}}"
	  loop_control: 
	    label: "{{item.hostname}}"
    ```
    
	* The task will loop through the ```device_uuid_cdguuid_list``` variable
	* For each item in the variable list, it will generate an API payload by using the Ansible template module
	* The template module mainly consist of two parts:
		* ```src``` - Jinja2 template source path
		* ```dest``` - destionation location to render the template output
	* Inspect this Jinja2 template ```ansible_project/roles/attach_device_to_cdg/templates/attach-cdg.j2```([content link](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/roles/attach_device_to_cdg/templates/attach-cdg.j2))
	* All the variables referenced in the template can be found in the ```device_uuid_cdguuid_list```variable. Here is an example variable:
	* 
	```
	    "device_uuid_cdguuid_list": [
	        {
	            "cdguuid": "d6cfd4ef-8f99-4ee4-a5d8-0c11543de5d1",
	            "device_uuid": "727cfcbf-8041-4ee3-87fb-b30d632bd596",
	            "hostname": "Node-1",
	            "ip": "198.19.1.1"
	        },
	        {
	            "cdguuid": "d6cfd4ef-8f99-4ee4-a5d8-0c11543de5d1",
	            "device_uuid": "7545ffcb-92f6-4c59-ac5f-d1d6129b78d5",
	            "hostname": "Node-2",
	            "ip": "198.19.1.2"
	        },
	        {
	            "cdguuid": "d6cfd4ef-8f99-4ee4-a5d8-0c11543de5d1",
	            "device_uuid": "38d39757-0445-48bc-8035-651873a29e89",
	            "hostname": "Node-3",
	            "ip": "198.19.1.3"
	        }
	    ]
	```
	
	* Example of the payloads```ansible_project/temp_folder/attach_device_to_cdg_api_payload```: [content link](https://github.com/schen1111/devwks-2100/tree/main/ansible_project/temp_folder/attach_device_to_cdg_api_payload)
	* Note that the API payload files are generated for reference only so you can see the payloads. The task that will invoke the attach device to CDG API will able to generate the payload on the fly without writing the payload to disk.
	* By using the template module, you can easily generate large amount of API payloads


#### Task 6: Attach Devices to CDG
1. Go to directory: 
```
cd /mnt/c/Users/Administrator/Downloads/devwks-2100/ansible_project
```
2. Execute the Ansible playbook to prepare for this task:
```
ansible-playbook cisco_live_prepare_for_attach_device_to_cdg_play.yml
```
3. Log into Crosswork Web GUI in the Windows VM: ```https://198.18.134.219:30603/#/cdg/inventory```. You can view devices that are currently attached to CDG by looking at the "Attached Device Count". There should be none at this point
	* Username: ```admin```
	* Password: ```C!sco12345```
4. Execute the below playbook to identify the VDG UUID for your instance ([content link](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/obtain_cdg_info_play.yml)):
```
ansible-playbook obtain_cdg_info_play.yml
```
5. (Optional) You can obtain the VDG UUID from below output. The Ansible play from the last step already updated ```ansible_project/global_variable.yml``` with the CDG UUID information for each device
```
cat temp_folder/obtain_cdg_info/cdg-info
```
5. Execute the below playbook to onboard devices ([content link](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/device-onboarding-play.yml)):
```
ansible-playbook attach_device_to_cdg_play.yml
```
5. Check the Crosswork Web GUI in the Windows VM for "Attached Device Count": ```https://198.18.134.219:30603/#/cdg/inventory```


<br/><br/>
### Use-case #3: Change the Admin State (up or down) of the Device in Crosswork
This use-case demonstrates how to leverage Ansible to change device admin state (up or down) in Crosswork.

#### Task 7: Understand Device Admin State Change
1. Device onboarded in Crosswork can be in few admin states: Up, Down, or Unmanaged
2. Some device setting might not able to be changed unless the admin state is in Down or Unmanaged state
3. You can define which devices are included in the admin state change by adding them or removing them in the ```ansible_project/global_variable.yml``` ([content link](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/global_variable.yml)). Devices section was converted from CSV to YAML. See Task 3 on how to convert CSV to YAML.
4. Inspect the Ansible tasks for **admin down**: ```ansible_project/roles/change_device_admin_state_down/tasks/main.yml```([content link](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/roles/change_device_admin_state_down/tasks/main.yml))
	* Task```invoke nodes query API to obtain device UUID for devices that are onboarded to CW``` will make an API call to obtain devices info which includes the UUID for each device.
5. Task```create API payloads for device admin down state```will generate device admin down state API payload from the variable ```getDeviceUUIDOutput``` using the Jinja2 template

	```yaml
	- name: create API payloads for device admin down state
	  template:
	    src: "../templates/admin-down.j2"
	    dest: "{{playbook_dir}}/temp_folder/device_admin_status_api_payload/device-admin-state-down-payload.json"
    ```

	* The template module mainly consist of two parts:
		* ```src``` - Jinja2 template source path
		* ```dest``` - destionation location to render the template output
	* Inspect this Jinja2 template ```ansible_project/roles/change_device_admin_state_down/templates/admin-down.j2```([content link](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/roles/change_device_admin_state_down/templates/admin-down.j2))

		```jinja2
		{
		    "data": [
		{% for item in getDeviceUUIDOutput.results %}
		        {
		            "uuid": "{{item.json.data[0].uuid}}",
		            "admin_state": "ROBOT_ADMIN_STATE_DOWN"
		        }
		{% if loop.revindex != 1 %}
		        ,
		{% endif %}
		{% endfor -%} 
		    ]
		}
		```
	* In this scenario, the API payload can contain many devices. It will do the loop within the template
	* For each item in the ```getDeviceUUIDOutput.results``` list, it will generate an API payload by adding the device UUID.
	* The "if statement" will add a comma after adding each item in the payload unless it is the last item in the list. This is to confirm the the standard json formatting
	* Here is an example of the generated payload ([content link](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/temp_folder/device_admin_status_api_payload/device-admin-state-down-payload.json)): 

		```json
		{
		    "data": [
		        {
		            "uuid": "38ae6962-23f5-4bea-8e64-c7d225536689",
		            "admin_state": "ROBOT_ADMIN_STATE_DOWN"
		        }
		        ,
		        {
		            "uuid": "ca4001a4-db5e-447e-9fd6-ec5532bb0704",
		            "admin_state": "ROBOT_ADMIN_STATE_DOWN"
		        }
		        ,
		        {
		            "uuid": "6212cc16-fc04-44d3-8ecf-c996e2bcb745",
		            "admin_state": "ROBOT_ADMIN_STATE_DOWN"
		        }
			]
		}
		```
	* Note that the API payload files are generated for reference only so you can see the payloads. The task that will invoke the change device admin state API will able to generate the payload on the fly without writing the payload to disk.
	* By using the template module, you can easily generate large amount of API payloads
6. Inspect the Ansible tasks for **admin up**: ```ansible_project/roles/change_device_admin_state_up/tasks/main.yml```([content link](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/roles/change_device_admin_state_up/tasks/main.yml)). 
	* The admin up playbook is similar to the admin down playbook. The primary difference in admin up is making one API call per device instead of combining all devices in a single API call when making device admin state change. There is no technical reason behind it. It was done to demonstrate a different method.
	* The below task is looping through ```getDeviceUUIDOutput``` variable to generate an API payload per device.
	
	```yaml
	- name: create API payload files for admin up device
	  template:
	    src: "../templates/admin-up.j2"
	    dest: "{{playbook_dir}}/temp_folder/device_admin_status_api_payload/{{item.json.data[0].host_name}}-admin-state-up-payload.json"
	  loop: "{{getDeviceUUIDOutput.results}}"
	  loop_control: 
	    label: "{{item.json.data[0].host_name}}"
    ```
	* There's no looping within the template itself ([content link](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/roles/change_device_admin_state_up/templates/admin-up.j2)):
	
	```json
	{
	    "data": [
	        {
	            "uuid": "{{item.json.data[0].uuid}}",
	            "admin_state": "ROBOT_ADMIN_STATE_UP"
	        }
	    ]
	}	
	```
	*  Here is an example of the generated payload ([content link](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/temp_folder/device_admin_status_api_payload/Node-1-admin-state-up-payload.json)):
	
	```json
	{
	    "data": [
	        {
	            "uuid": "38ae6962-23f5-4bea-8e64-c7d225536689",
	            "admin_state": "ROBOT_ADMIN_STATE_UP"
	            
	        }
	    ]
	}
	```


#### Task 8: Perform Device Admin State Change
1. Go to directory: 
```
cd /mnt/c/Users/Administrator/Downloads/devwks-2100/ansible_project
```
2. Log into Crosswork Web GUI in the Windows VM: ```https://198.18.134.219:30603/#/inventory/overview```. You can view devices that are currently onboarded to Crosswork and their admin status
	* Username: ```admin```
	* Password: ```C!sco12345```
3. The devices listed in ```ansible_project/global_variable.yml```([content link](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/global_variable.yml)) will be have their admin state changed.
4. Execute the below playbook to **admin down** devices ([content link](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/change_device_admin_state_down_play.yml)):
```
ansible-playbook change_device_admin_state_down_play.yml 
```
5. Check the Crosswork Web GUI in the Windows VM for devices admin status: ```https://198.18.134.219:30603/#/inventory/overview```
6. Execute the below playbook to **admin up** devices ([content link](https://github.com/schen1111/devwks-2100/blob/main/ansible_project/change_device_admin_state_up_play.yml)):
```
ansible-playbook change_device_admin_state_up_play.yml
```
7. Check the Crosswork Web GUI in the Windows VM for devices admin status: ```https://198.18.134.219:30603/#/inventory/overview```





