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
There are a couple of ways where we can share roles:
1. Dependencies through web service artifact
    
    Example:
    
    In the `roles-sharing` project,
    ```bash
    cd roles-artifact-server
    ./start-server
    ```
    In another console inside `roles-sharing`,
    ```bash
    ansible-galaxy install --force -r requirements.yml
    ```
    This will install the role to a default shared location. To install it into the roles directory of the current project,
    ```bash
    ansible-galaxy install --force -r requirements.yml -p roles
    ```
    Run the playbook,
    ```bash
    nsible-playbook -i localhost, --connection=local roles-sharing.yml -v
    ```

    Advantages:
      + Able to share code with versioning (somewhat)
    
    Disadvantages:
      - Need to build pipeline to publish role artifact to web service
      - If not installed directly into project roles directory (and checked into vcs), versioning will conflict with other projects

2. Dependencies through git
    
    Advantages:
      + Able to leverage git for versioning (somewhat)

    Disadvantages:
      - Need to setup a separate git repo for each role
      - If not installed directly into project roles directory (and checked into vcs), versioning will conflict with other projects

3. Setting roles_path to point to "roles" git repo

    Advantages:
      + Centralised place to manage roles
    
    Disadvantages:
      - No versioning, latest version checked out used
      - Need to coordinate checking out of repo when using playbook on CI
      - Need to ensure roles_path is consistently set

References:
- https://docs.ansible.com/ansible/latest/reference_appendices/galaxy.html#installing-multiple-roles-from-a-file

## Misc Ways/ Ideas to Improve Playbook
1. Prefer idempotent ansible modules
2. Split up playbooks that does Configuration Management vs Deployment
3. Breakup Larger plays into smaller plays (100 line guideline?)

## Ansible: The Bad Parts/ Pitfalls
1. Lack of namespaces which causes complexity in knowing which variable is overwritten
  - See [Variable Precedence](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable)
  - Problematic for roles which on first sight seems like it should namespace its own variables just like modules in other programming languages
  - Possible way to reduce the pain is to prefix variable with the name (or some alias) of the role
    * E.g. `nginx_dir` instead of just `dir`
    * Brings its set of issues like redundancy and is more of a workaround
2. Can't limit priviledge escalation to certain commands
  - Given that it is common for Security to follow the principle of least priviledge, seems like a huge limitation
  - We end up using the shell or command module to workaround this limitation, which kind of defeats the purpose of using ansible for idempotent commands
  - See for details: https://docs.ansible.com/ansible/latest/user_guide/become.html#can-t-limit-escalation-to-certain-commands

## Tools
- [ansible-lint](https://github.com/ansible/ansible-lint)

## Possible Ideas for Testing
- Spin up docker instances for test
- [molecule](https://github.com/ansible/molecule)
- [goss](https://github.com/aelsabbahy/goss)

## Fun with Cows
From [Ansible FAQ](https://docs.ansible.com/ansible/latest/reference_appendices/faq.html#how-do-i-disable-cowsay):
> If cowsay is installed, Ansible takes it upon itself to make your day happier when running playbooks.

If you don't like cows, try `ANSIBLE_COW_SELECTION=tux`.

Or for a bit of surprise, try `ANSIBLE_COW_SELECTION=random`.

To see what cows are available: `cowsay -l`

Unfortunately...
> If you decide that you would like to work in a professional cow-free environment, you can either uninstall cowsay, set nocows=1 in ansible.cfg, or set the ANSIBLE_NOCOWS environment variable: `export ANSIBLE_NOCOWS=1`

Reference: https://michaelheap.com/cowsay-and-ansible/