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


## Misc Ways to Improve Playbook

## Tools

## Sharing of Roles via Git

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