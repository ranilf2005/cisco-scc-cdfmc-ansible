## Use lists in group_vars/all.yml, then loop through them.

1. Update group_vars/all.yml

```yaml
bulk_host_objects:
  - name: "ansible-host-192.168.78.35"
    value: "192.168.78.35"
  - name: "ansible-host-192.168.78.36"
    value: "192.168.78.36"
  - name: "ansible-host-192.168.78.37"
    value: "192.168.78.37"

bulk_access_rules:
  - name: "Block Host 192.168.78.35"
    action: "BLOCK"
    source_object_name: "ansible-host-192.168.78.35"

  - name: "Block Host 192.168.78.36"
    action: "BLOCK"
    source_object_name: "ansible-host-192.168.78.36"

  - name: "Block Host 192.168.78.37"
    action: "BLOCK"
    source_object_name: "ansible-host-192.168.78.37"
```

## 2. Create bulk objects playbook

Create:

```bash
nano playbooks/03_bulk_create_objects.yml
```

Paste:

---

```yaml
- name: Bulk create host objects
  hosts: localhost
  gather_facts: false

  vars_files:
    - ../group_vars/all.yml

  tasks:
    - name: Create host objects
      uri:
        url: "{{ base_url }}/domain/{{ domain_uuid }}/object/hosts"
        method: POST
        headers: "{{ common_headers }}"
        body_format: json
        body:
          name: "{{ item.name }}"
          type: "Host"
          value: "{{ item.value }}"
        return_content: true
        status_code:
          - 201
          - 202
          - 400
      loop: "{{ bulk_host_objects }}"
      register: object_results

    - name: Show object creation results
      debug:
        msg: "{{ item.item.name }} -> HTTP {{ item.status }}"
      loop: "{{ object_results.results }}"
```

Run:

```bash
ansible-playbook -i inventory.yml playbooks/03_bulk_create_objects.yml
```

## 3. Create bulk access rules playbook

Create:

```bash
nano playbooks/04_bulk_create_access_rules.yml
```

Paste:

---
```yaml
- name: Bulk create access rules
  hosts: localhost
  gather_facts: false

  vars_files:
    - ../group_vars/all.yml

  tasks:
    - name: Get all host objects
      uri:
        url: "{{ base_url }}/domain/{{ domain_uuid }}/object/hosts?expanded=true"
        method: GET
        headers: "{{ common_headers }}"
        return_content: true
        status_code: 200
      register: host_lookup

    - name: Create access rules
      uri:
        url: "{{ base_url }}/domain/{{ domain_uuid }}/policy/accesspolicies/{{ access_policy_id }}/accessrules"
        method: POST
        headers: "{{ common_headers }}"
        body_format: json
        body:
          name: "{{ item.name }}"
          type: "AccessRule"
          action: "{{ item.action }}"
          enabled: true
          sourceNetworks:
            objects:
              - id: "{{ (host_lookup.json['items'] | selectattr('name', 'equalto', item.source_object_name) | list | first).id }}"
                name: "{{ item.source_object_name }}"
                type: "Host"
        return_content: true
        status_code:
          - 201
          - 202
          - 400
      loop: "{{ bulk_access_rules }}"
      register: rule_results

    - name: Show rule creation results
      debug:
        msg: "{{ item.item.name }} -> HTTP {{ item.status }}"
      loop: "{{ rule_results.results }}"
```

Run:

```yaml
ansible-playbook -i inventory.yml playbooks/04_bulk_create_access_rules.yml
4. Verify rules
curl -sk -X GET \
  "{{YOUR_BASE_URL}}/domain/{{YOUR_DOMAIN_UUID}}/policy/accesspolicies/{{YOUR_POLICY_ID}}/accessrules?expanded=true" \
  -H "Authorization: Bearer $CDFMC_API_TOKEN" \
  -H "Accept: application/json" | jq '.items[] | {name, action, id}'
```

For your current AUS tenant, use:

```bash
curl -sk -X GET \
  "https://cisco-spark-cisco-security-platform--sxroua.app.aus.cdo.cisco.com/api/fmc_config/v1/domain/e276abec-e0f2-11e3-8169-6d9ed49b625f/policy/accesspolicies/0AF4F6DE-033D-0ed3-0000-004294969478/accessrules?expanded=true" \
  -H "Authorization: Bearer $CDFMC_API_TOKEN" \
  -H "Accept: application/json" | jq '.items[] | {name, action, id}'
```

Use this order:

```bash
ansible-playbook -i inventory.yml playbooks/03_bulk_create_objects.yml
ansible-playbook -i inventory.yml playbooks/04_bulk_create_access_rules.yml
```
