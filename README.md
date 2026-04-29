# Cisco SCC–cdFMC Automation (Ansible) – Lab & Testing Guide

> [!WARNING]
> ## ⚠️ IMPORTANT – READ BEFORE USE
> This repository contains **lab validation scripts and proof-of-concept automation only**.
>
> - **NOT intended for production use**
> - **DO NOT execute in live/production environments**
> - Scripts are provided for **learning, testing, and concept validation purposes only**
>
> These examples demonstrate how Cisco Secure Cloud Control (SCC) / cdFMC APIs can be used with Ansible.
> They are **generic samples** and **must be adapted, validated, and tested** based on your own environment,
> security policies, and operational requirements.
>
> ⚡ **Use at your own risk**  
> The author assumes **no responsibility or liability** for any impact, damage, or misconfiguration
> resulting from the use of these scripts.
>
> Always follow your organization's **change control, security, and validation processes** before applying any automation.

---
<br>

## 📌 Overview

This guide demonstrates a **basic automation workflow** for Cisco **SCC / cdFMC** using:

- REST APIs
- API Token Authentication
- Ansible Playbooks

The objective is to:
- Validate API connectivity
- Retrieve policy information
- Create objects
- Create access rules

---
<br>

## 🔐 Step 1 – Create SCC/CDO API Token

1. Log in to your **Cisco Secure Cloud Control (CDO)** tenant  
2. Create an **API user/token**
3. Copy the generated token securely

To use an API token and connect to the Cloud-Delivered Firewall Management Center (CDFMC) for REST API management, follow these steps:

Generate an API Token

Navigate to Settings > User Management (or Administration > API User Management depending on your workflow).
Under the desired account, select Generate API Token.
Copy and securely store the API token. This token is a long-lived bearer token and must be included in the authorization header of your REST API calls[1][2].
REST API Authentication

Use the token in the HTTP Authorization header for all REST API requests.
Example using curl:
```bash
curl 'https://<cdfmc-server>:<port>/api/fmc_config/v1/domain/<domainUUID>/object/hosts'
--header 'Authorization: Bearer <api_token>' ```
```

The token replaces the need for username and password authentication in each request.
Best Practices

Use separate accounts for API and UI access; do not use the same credentials for both interfaces simultaneously.
Assign only the necessary privileges to API users.
Always validate and sanitize content received from the server, especially JSON payloads[3].


---
<br><br>

## 🔑 Step 2 – Export API Token

```bash
export CDFMC_API_TOKEN='<YOUR_NEW_API_USER_TOKEN>'
```

![export](/images/export.png)


Verify:

```bash
echo $CDFMC_API_TOKEN
```

![export](/images/echo.png)
<br><br>

## 🌐 Step 3 – Test Base Connectivity

```bash
curl -sk -X GET \
  "https://api.apj.security.cisco.com/firewall/v1/inventory/devices" \
  -H "Authorization: Bearer $CDFMC_API_TOKEN" \
  -H "Accept: application/json"
```

![export](/images/curl1.png)
<br>

Extract the following details:
- domain_uuid: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
- device_record_uuid: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
- current_access_policy_id: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
- base_url: https://api.apj.security.cisco.com/firewall
<br><br>


## 📊 Step 4 – Retrieve Access Policy Details

```bash
curl -sk -X GET \
  "https://api.apj.security.cisco.com/firewall/v1/cdfmc/api/fmc_config/v1/domain/e276abec-e0f2-11e3-8169-6d9ed49b625f/policy/accesspolicies" \
  -H "Authorization: Bearer $CDFMC_API_TOKEN" \
  -H "Accept: application/json"
```

![export](/images/curl2.png)
<br><br>


## ✅ Step 5 – Validate Connectivity (Ansible)
```yaml
ansible-playbook -i inventory.yml playbooks/00_test.yml
```

![export](/images/test1.png)
<br><br>

## 🧱 Step 6 – Create Object
```yaml
ansible-playbook -i inventory.yml playbooks/01_create_object.yml
```
<br><br>

## 🔐 Step 7 – Create Access Rule
```yaml
ansible-playbook -i inventory.yml playbooks/02_create_access_rule.yml
```
<br><br>

## ▶️ Recommended Test Execution Order
```shell
export CDFMC_API_TOKEN='<YOUR_CDO_API_TOKEN>'
ansible-playbook -i inventory.yml playbooks/00_test.yml
ansible-playbook -i inventory.yml playbooks/01_create_object.yml
ansible-playbook -i inventory.yml playbooks/02_create_access_rule.yml
```
<br><br>

## 🧠 Key Notes
All scripts are modular and reusable
Modify playbooks based on:
Your use case
Your policy structure
Your object naming standards
Always validate API responses before proceeding to the next step
<br><br>

## 📎 Disclaimer

This project is intended to:

Demonstrate automation capabilities
Provide guidance on API integration
Showcase tested concepts in a lab environment
It is not a production-ready solution.

