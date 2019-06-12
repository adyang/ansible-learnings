# Ansible Learnings

## Development Setup


## Useful Ad-Hoc Commands
Check if servers are up:
```bash
ansible host-pattern -i inventory-file -m ping
```

Running commands on hosts:
```bash
ansible host-pattern -i inventory-file -m shell -a 'echo hi'
ansible host-pattern -i inventory-file -m shell -a 'sudo su - user -c "whoami"'
```

Templating:
```bash
ansible localhost -i inventory-file --connection=local -m template -a 'src=templates/example.json.j2 dest=example.json'
cat example.json
```

Copying file(s) to host(s):
```bash
ansible host-pattern -i inventory-file -m copy -a 'src=/file/on/host dest=/location/on/control/machine'
```

Downloading file(s) from host(s):
```bash
ansible host-pattern -i inventory-file -m fetch -a 'src=/file/on/host dest=/directory/on/control/machine'
cat /directory/on/control/machine/192.168.X.X/file/on/host

ansible 192.168.X.X -i inventory-file -m fetch -a 'src=/file/on/host dest=/directory/on/control/machine/ flat=yes'
cat /directory/on/control/machine/host
```
For the flat case, the files will overwrite each other if used from more than 1 host (thus it is best to use it on a single host) and note the trailing slash for the `dest` directory.

References
- https://stackoverflow.com/questions/35407822/how-can-i-test-jinja2-templates-in-ansible
- https://docs.ansible.com/ansible/latest/modules/template_module.html#template-module
- https://docs.ansible.com/ansible/latest/modules/copy_module.html#copy-module
- https://docs.ansible.com/ansible/latest/modules/fetch_module.html#fetch-module

## Roles
Roles are a way to organise/ package a set of tasks to improve reusability and sharing.

### Role Initialisation
We can initialise a role using:
```bash
ansible-galaxy init --init-path=roles role_name
```
Alternatively, we can manually create the directories and omit the ones that we do not need.

### What is in a role?
Most directories will contain a main.yml (except files and templates) and their purpose is as follows:
- tasks: tasks in the main.yml file will be executed as part of the role, other yml files can be included/ imported from main.yml
- defaults: default variables will automatically be loaded
- vars: variables that will automatically be loaded; has higher precedence than defaults, in particular, inventory variables cannot overwrite them
- files: allows reference to files inside directly without the need to state their path (e.g. in copy/ script module)
- templates: similar to files but for templates (e.g. in template module)
- handlers: will make all handlers from main.yml available to role and play
- meta: meta data such as dependencies; required for sharing via Ansible Galaxy

### How to use them?
1. Classic way
    ```yaml
    - hosts: localhost
      roles:
        - roleOne
        - role: roleOne
          vars:
            varFour: fromRoleParam
    ```
2. Ansible 2.4 and above
    ```yaml
    - hosts: localhost
      tasks:
        - import_role:
            name: roleThree
          vars:
            varFive: fromRoleImport
    ```
In the `roles-example` directory, run ansible-playbook to see output:
```bash
ansible-playbook -i localhost, --connection=local roles-example.yml -v
```

### Benefits
+ Able to break down plays into smaller units
+ Easier to try out smaller units -> Faster feedback
+ Reusability
+ Possibility of sharing code (See [Sharing of Roles](#sharing-of-roles))

### Disadvantages
- Fragmented structure, need to refer to several files to understand play
- No true namespacing -> unwanted variable overwrites (See [Ansible: The Bad Parts](#ansible-the-bad-parts))
- Awkward sharing system

References
- https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html
- https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse.html

## Sharing of Roles

## Misc Ways to Improve Playbook

## Ansible: The Bad Parts

## Tools

## Possible Ideas for Testing

## Fun with Cows
From [Ansible FAQ](https://docs.ansible.com/ansible/latest/reference_appendices/faq.html#how-do-i-disable-cowsay):
> If cowsay is installed, Ansible takes it upon itself to make your day happier when running playbooks.

If you don't like cows, try `ANSIBLE_COW_SELECTION=tux`.

Or for a bit of surprise, try `ANSIBLE_COW_SELECTION=random`.

To see what cows are available: `cowsay -l`

Unfortunately...
> If you decide that you would like to work in a professional cow-free environment, you can either uninstall cowsay, set nocows=1 in ansible.cfg, or set the ANSIBLE_NOCOWS environment variable: `export ANSIBLE_NOCOWS=1`

Reference: https://michaelheap.com/cowsay-and-ansible/