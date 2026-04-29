## *** VERY IMPPORTANT ***
This document explain very simple testing of Cisco SCC-cdFMC using Automation Ansible. This test will use API token.  
<br>

## Step 1 - Create your SCC-CDO tenant API user with token
Login to your SCC CDO tenanant and create your API user token  
<br><br>


## Step 2 - Exporting API Token Key

```bash
export CDFMC_API_TOKEN='<YOUR_NEW_API_USER_TOKEN>'
```
```bash
echo $CDFMC_API_TOKEN
```
<br>

## Step 3 - Test Base Connectivity
```curl
curl -sk -X GET \
  "https://api.apj.security.cisco.com/firewall/v1/inventory/devices" \
  -H "Authorization: Bearer $CDFMC_API_TOKEN" \
  -H "Accept: application/json"
```

Extract following details from above results
- domain_uuid: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
- device_record_uuid: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
- current_access_policy_id: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
- base_url: "https://api.apj.security.cisco.com/firewall"
<br>


## Step 4 - Receving Access Policy Details
```curl
curl -sk -X GET \
  "https://api.apj.security.cisco.com/firewall/v1/cdfmc/api/fmc_config/v1/domain/e276abec-e0f2-11e3-8169-6d9ed49b625f/policy/accesspolicies" \
  -H "Authorization: Bearer $CDFMC_API_TOKEN" \
  -H "Accept: application/json"
```
<br>


## Step 5 - Validate connectivity
```yaml
ansible-playbook -i inventory.yml playbooks/00_test.yml
```
<br>

## Step 6 - Create Object
```yaml
ansible-playbook -i inventory.yml playbooks/01_create_object.yml
```
<br>

## Step 7 - Create Access Rule
```yaml
ansible-playbook -i inventory.yml playbooks/02_create_access_rule.yml
```
<br>


## Testing Order

```shell
export CDFMC_API_TOKEN='<YOUR_CDO_API_TOKEN>'

ansible-playbook -i inventory.yml playbooks/00_test.yml
ansible-playbook -i inventory.yml playbooks/01_create_object.yml
ansible-playbook -i inventory.yml playbooks/02_create_access_rule.yml
```


