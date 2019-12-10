---
title: "4.1 - Ansible Playbooks - Basics"
date: 2019-11-29T11:02:05+06:00
weight: 41
---

In this lab we’ll get used to writing and running ansible playbooks.

### Task 1

Create a playbook `webserver.yml` which does the following:

- Install `httpd` on the nodes in the `web` group.
- Start `httpd` and ensure the service starts on boot. Ensure that the firewall is also started and enabled.
- Ensure port 80 is open on the firewall.

{{% notice tip %}}
Check what the options "immediate" and "permanent" of the service module mean and do.
{{% /notice %}}

- Run the playbook. After completion test if the `httpd.service` is running and enabled on node1.

### Task 2

- Create a folder `inventory` and move your inventory `hosts` there.
- Configure ansible to use `/home/ansible/techlab/hosts` as the default inventory.
  Do this using a configuration file in the `/home/ansible/techlab/` directory.
- Run the playbook again without using the `-i` flag to see if the configuration works.

### Task 3

- Add intentionally errors to your plabook and have a look at the output. You should get a feeling for errormessages.
- add a wrong intendation. Remember that this is a common mistake!
- add a wrong parameter name
- remove the mistakes

### Task 4

- Crete a playbook `tempfolder.yml`
- The playbook `tempfolder.yml`  should create a temporary folder `/root/tempfolder` on all servers in the group `db`
- The folder has to have the sticky bit set, so that only owner of the content (or root) can delete the files
- Run the playbook and then check if the sticky bit was set using an ad hoc command

## Solutions

{{% collapse solution-1 "Solution 1" %}}

Below is a possible solution for your playbook:

```yaml
---
- hosts: web
  become: yes
  tasks:
    - name: install httpd
      yum:
        name: httpd
        state: installed
    - name: start and enable httpd
      service:
        name: httpd
        state: started
        enabled: yes
    - name: start and enable firewalld
      service:
        name: firewalld
        state: started
        enabled: yes
    - name: open firewall for http
      firewalld:
        service: http
        state: enabled
        permanent: yes
        immediate: yes
```

Run your playbook with:

```bash
$ ansible-playbook -i hosts webserver.yml
```

Check `httpd.service` on group `web`:

```bash
$ ansible web -b -a "systemctl status httpd"
```
{{% /collapse %}}

{{% collapse solution-2 "Solution 2" %}}

Copy the default ansible.cfg to your directory:

```bash
$ mkdir inventory; mv hosts inventory/hosts
$ mv /home/ansible/techlab/hosts /home/ansible/techlab/inventory/
$ cp /etc/ansible/ansible.cfg /home/ansible/techlab/
```

Edit your `ansible.cfg` file. Uncomment and edit the first "inventory" entry to use your file:

```bash
[defaults]
# some basic default values...
inventory      = /home/ansible/techlab/inventory/hosts # <-- edit this line
#library        = /usr/share/my_modules/
```

```bash
$ ansible-playbook webserver.yml
PLAY [web] ***********************************************************************

TASK [Gathering Facts] ***********************************************************
ok: [node1]

TASK [install httpd] *************************************************************
ok: [node1]
```
{{% /collapse %}}

{{% collapse solution-3 "Solution 3" %}}

Wrong intendation:

```yaml
---
- hosts: web
  become: yes
  tasks:
    - name: install httpd
      yum:
      name: httpd	      # <-- wrong intendation
      state: installed  # <-- wrong intendation
```

Wrong parameter name:

```yaml
---
- hosts: web
  become: yes
  tasks:
    - name: install httpd
      yum:
        name: httpd
        state: installed
        enabled: yes     # <-- doesn't exist for yum module
```

{{% /collapse %}}

{{% collapse solution-4 "Solution 4" %}}
```bash
$ cat tempfolder.yml
---
- hosts: db
  become: yes
  tasks:
    - name: create temp folder with sticky bit set
      file:
        dest: /root/tempfolder
        mode: "01755"
        state: directory

$ ansible-playbook tempfolder.yml
$ ansible db -b -a "ls -lah /root/"

```
{{% notice note %}}
`ansible-doc file` doesn't provide any information about setting special permissions like sticky bit. Remember to use a leading 0  and then your permissions.
{{% /notice %}}

{{% /collapse %}}
