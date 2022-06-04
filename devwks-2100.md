# DevNet Workshop - DEVWKS-2100

# Build Ansible Plays to Automate Crosswork API Invocations


### Sanka Chen, CCIE #49686, Technical Leader, Cisco Systems

### Weigang Huang, CCIE #49667, Customer Delivery Software Architect, Cisco Systems


## Workshop guide

### Use Case Summary
This workshop provides walkthrough for few use cases to automate Cisco Crosswork API by using Ansible. Participants can leverage the provided use cases to start their Crosswork API automation journey.

#### Usecase #1: Device onboarding
#### Usecase #2: Attach devices to CDG (Crosswork Data Collector)
#### Usecase #3: Change the admin state (up or down) of the device in Crosswork

<br/><br/>
### API Authentication (Common Across all Use Cases)
In order to invoke any Crosswork API, you must first obtain a JWT (JSON Web Token). The JWT needs to be in every HTTP header of API calls using the Bearer Authorization. This is a two step process. 

1. Request a TGT (Ticket Granting Ticket) using your Crosswork GUI credential 
2. Request a JWT using the TGT in step 1

#### Task 1: Perform Authentication using Postman
1. Open Postman (shortcut in desktop)
2. Expand this collection "DEVWKS-2100"
3. Expand CW Authentication folder. You should see two post requests
4. Select "cw-api-get-ticket-step1" reqeust and hit send. You should see a TGT in the response
5. Select "cw-api-get-jwt-step2" request and hit send. You should see a JWT in the response

#### Task 2: Perform Authentication using Ansible
1. Open Ubuntu (shortcut in desktop)
2. Go to dir:
```
cd /mnt/c/Users/Administrator/Downloads/devwks-2100
```
3. Inspect the playbook:
```
cat roles/get_crosswork_authentication/tasks/main.yml
``` 
4. Execute this playbook:
```
ansible-playbook get_crosswork_authentication_play.yml
```
5. You should see the the TGT and JWT in the debug output from the Ansible play


<br/><br/>
### Usecase #1: Device onboarding
This usecase demonstrates how to leverage Ansible to onboard devices to Crosswork.

1. Most of you will probably get a list of devices to onboard via an Excel sheet. One of the easier ways for Ansible to consume the data is to convert the Excel into Yaml file. You can do it fairly easily using online tools
2. (Not required for this lab) - Covert Excel to CSV
3. (Not required for this lab) - Covert CSV to YAML by using online converter. Converter can be found via search engines
4. (Not required for this lab) - Use online YAML beautifier to properly format the converted YAML file. YAML beautifierc an be found via search engines.
5. Inspect the global_variable.yml to see the coverted CSV under the devices section
```
cat global_variable.yml
```
6. Inpsect the Ansible tasks to onboard the devices. The task will generate API payload with each device's information from the global_variable.yml using the Jinja2 template.
```
cat roles/device_onboarding/tasks/main.yml
```
7. d



