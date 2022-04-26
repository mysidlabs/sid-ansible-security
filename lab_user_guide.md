# Ansible Security Immersion Day Lab Outline
## Initial Setup
### Connect to your jump box.
Connect to your jump host where ### is replaced by your user number:
```bash
>>  ssh siduser###@siduser###.jump.mysidlabs.com
Password: Chi1962!
```
You should then see the following:
```bash
Welcome to the CDW/Sirius Red Hat Immersion Day lab environment
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
Login Succeeded!
Cloning into 'sid-ansible-security'...
remote: Enumerating objects: 125, done.
remote: Counting objects: 100% (125/125), done.
remote: Compressing objects: 100% (69/69), done.
remote: Total 125 (delta 44), reused 111 (delta 32), pack-reused 0
Receiving objects: 100% (125/125), 22.19 KiB | 11.09 MiB/s, done.
Resolving deltas: 100% (44/44), done. 
siduser270@jump:~$ 
```
### Review jump environment
Installed tools:
  * `git` - command line tool to interact with git source control repositories.
  * `podman` - a name-spaced drop in replacement for the docker cli.
  * `nano` - a simple text editor.
  * `vim` - a powerful text editor.
  * `ansible-builder` - a command line wrapper to podman to build execution environments.
  * `ansible-navigator` - a powerful replacement to the old ansible-playbook command.
  * `ansible-vault` - a command line encryption tool to protect secrets for ansible-playbooks.
  * `tree` - a command line utilitiy to show recursive directory listings in a visual format.

    We will be editing text files throught the course of this lab.  Both `nano` and `vim` are available.  If you are inexperienced with `vim`,  `nano` will be the better choice, as it behaves more like a basic text editor.


## Lab 1 - `ansible-navigator` and execution environments
### Review the project structure
```bash
siduser270@jump:~$ cd sid-ansible-security/
siduser270@jump:~/sid-ansible-security$ ls
amp-mitigation-play.yml  ansible-navigator.yml     create-soc-ticket.yml  ee              group_vars  lab-key        README.md  simulate-amp-securex-POST-request
ansible.cfg              create-security-rule.yml  dev-data               filter_plugins  inventory   navigator-log  roles
siduser270@jump:~/sid-ansible-security$ 

```
### Using Ansible Vault with ansible-navigator
[https://github.com/ansible/ansible-navigator/blob/main/docs/faq.md]

There are two methods to expose a vault password for use with ansible-navigator:
1. Store the vault password securely on the local files system:
    ```bash
    $ touch ~/.vault_password
    $ chmod 600 ~/.vault_password
    # The leading space here is necessary to keep the command out of the command history
    $  echo my_password >> ~/.vault_password
    # Link the password file into the current working directory
    $ ln ~/.vault_password .
    # Set the environment variable to the location of the file
    $ export ANSIBLE_VAULT_PASSWORD_FILE=.vault_password
    ```
2. Store the vault password in an environment variable
    ```bash
    $ touch ~/.vault_password.sh
    $ chmod 700 ~/.vault_password.sh
    $ echo -e '#!/bin/sh\necho ${ANSIBLE_VAULT_PASSWORD}' >> ~/.vault_password.sh
    # Link the password file into the current working directory
    $ ln ~/.vault_password.sh .
    # The leading space here is necessary to keep the command out of the command history
    # by using an environment variable prefixed with ANSIBLE it will automatically get passed
    # into the execution environment
    $  export ANSIBLE_VAULT_SECRET=my_password
    # Set the environment variable to the location of the file
    $ ANSIBLE_VAULT_PASSWORD_FILE=.vault_password.sh
    ```

Execute the following command to decrypt the group_vars/all.yml file:
```bash
    siduser101@jump:~/sid-ansible-security$ ansible-vault decrypt group_vars/all.yml 
    [DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current 
    version: 3.6.8 (default, Mar 18 2021, 08:58:41) [GCC 8.4.1 20200928 (Red Hat 8.4.1-1)]. This feature will be removed 
    from ansible-core in version 2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in 
    ansible.cfg.
    Vault password: Ger1974!
    Decryption successful
    siduser101@jump:~/sid-ansible-security$
```
### Use `podman`, `ansible-builder` and `ansible-navigator` to work with execution environments
Type the following at the command line:
```bash
siduser270@jump:~/sid-ansible-security$ podman images
REPOSITORY  TAG         IMAGE ID    CREATED     SIZE
siduser270@jump:~/sid-ansible-security$ 
```
Notice that there are currently no local images.
Type the `ansible-navigator` command, first making sure that you are in the root of the sid-ansible-security project:
```bash
siduser270@jump:~/sid-ansible-security$ ansible-navigator
---------------------------------------------------------------------------
Execution environment image and pull policy overview
---------------------------------------------------------------------------
Execution environment image name:  ghcr.io/mysidlabs/sid-security-ee:latest
Execution environment image tag:   latest
Execution environment pull policy: missing
Execution environment pull needed: True
---------------------------------------------------------------------------
Updating the execution environment
---------------------------------------------------------------------------
Trying to pull ghcr.io/mysidlabs/sid-security-ee:latest...
Getting image source signatures
Copying blob 2b28c7d19219 done  
Copying blob ab4052451ca3 done  
Copying blob aadba1834798 done  
Copying blob 1a80e05e2ae5 done  
Copying blob 6480d4237baa done  
Copying blob 4bf9522da935 done  
Copying blob 01831b54c91c done  
Copying blob 2cc40fcd238a done  
Copying blob 1352d2306a8c done  
Copying blob fb99cf4e728e done  
Copying blob ad10175f8b05 done  
Copying blob 5d1b0663c816 done  
Copying blob 688f93a4c9a9 done  
Copying blob a371d1a50b3c done  
Copying blob 15efb5cc51b2 done  
Copying blob d39a364c4e6f done  
Copying blob f84f34d1d332 done  
Copying blob 6bca7fed2387 done  
Copying config 7d187fedb8 done  
Writing manifest to image destination
Storing signatures
7d187fedb8ea9d3df35713baed9f00d7f8bb88549af5f788e8288fe9da7b8cd9
siduser270@jump:~/sid-ansible-security$
```
Since this is our first execution of ansible-navigator it has pulled the execution environment specified in the `ansible-navigator.yml` file which is the configuration file for this tool.

> :warning: Follow along with the instructor as he reviews the `ansible-navigator.yml` config and also looks at some of the sub-commands of the tool.


Use `ansible-navigator` to inspect new image:
```bash
siduser101@jump:~/sid-ansible-security$ ansible-navigator images
```

```
  NAME                             TAG      EXECUTION ENVIRONMENT	CREATED         SIZE
0│sid-security-ee (primary)        latest                    True	2 months ago    681 MB






^f/PgUp page up    ^b/PgDn page down    ↑↓ scroll    esc back    [0-9] goto    :help help

```

> :warning: Again, follow along with the instructor as he explores `sid-security-ee` execution environment.

### Review execution environments
1. Review ee project files
1. Review [`podman`](https://https://podman.io/) vs. `docker`
1. Review [`ansible-buider`](https://www.ansible.com/blog/introduction-to-ansible-builder)
1. Build sec-sid-mitigation-ee

> :warning: **OPTIONAL** building this execution environment is optional.  It will take a couple of minutes to complete.
```bash
    ansible-builder build --tag sec_sid_ee
```
### Use `podman` to list the local images:
```bash
siduser101@jump:~/sid-ansible-security$ podman images
REPOSITORY                                 TAG         IMAGE ID      CREATED            SIZE
<none>                                     <none>      cc8028f10813  About an hour ago  907 MB
<none>                                     <none>      a04cc3797cca  About an hour ago  913 MB
<none>                                     <none>      774fabe4d536  About an hour ago  881 MB
<none>                                     <none>      0643cb081bc6  About an hour ago  896 MB
localhost/sid-security-ee                  latest      9ad85776c89f  About an hour ago  908 MB
<none>                                     <none>      ee55b78317df  About an hour ago  791 MB
<none>                                     <none>      c5809904a915  About an hour ago  891 MB
quay.io/ansible/ansible-runner             latest      912ba7432e89  7 hours ago        876 MB
quay.io/ansible/ansible-builder            latest      35d2481da9e9  8 hours ago        769 MB
ghcr.io/mysidlabs/sid-security-ee          latest      7d187fedb8ea  2 months ago       681 MB
quay.io/ansible/ansible-navigator-demo-ee  0.6.0       e65e4777caa3  5 months ago       1.35 GB
```


## Lab 2 - Execute Plays with `ansible-navigator`
1. The instructor will log into the lab PaloAlto firewall and review the existing policies.
1. Edit `create-security-rule.yml` and modify rule_name key to:
    ```yaml
        rule_name: "siduser###- Block amp_{{ item.detection_id }}"
    ```
1. Review and then execute the `create-security-rule.yml` with `ansible-navigator`
    ```bash
    ansible-navigator run create-security-rule.yml --extra-vars "@dev-data/amp_single_event.json" 
    ```
1. Instructor will show rules added to Palo Alto NGFW
1. Edit `create-soc-ticket.yml` and modify summary key to:
    ```yaml
        summary: "siduser### - AMP: {{item.event_type}}: RHAP Automated mitigation"
    ```
1. Execute the `create-soc-ticket.yml` playbook with `ansible-navigator`
    ```bash
    ansible-navigator run create-soc-ticket.yml --extra-vars "@dev-data/amp_single_event.json" 
    ```
1. Instructor will show tickets added to JIRA project
1. Edit `roles/paloalto/tasks/main.yml` `rule_name:` key and `roles/jira/tasks/main.yml` `summary:` key to include your siduserID.
1. Execute `amp-mitigation-play.yml` with `ansible-navigator`:
    ```bash
    ansible-navigator run amp-mitigation-play.yml --extra-vars "@dev-data/amp_single_event.json" 
    ```


## Lab 3 - Automation controller and the AC REST api
1. Login to https://rhap2.mysidlabs.com
    * user: siduser###
    * password: Chi1962!
1. Review execution environments in AC.
1. Create project for individual repos.
1. Review registering external applications.
1. Review generating oauth2 token.
1. Create `amp-mitigation-play.yml` template.
1. Create workflow template for amp-mitigation:
    * Add project refresh node.
    * Add template node.
    * save
1. Review the AC REST API: `https://rhap2.mysidlabs.com/api/v2/`
    1. Look at the `/api/v2/workflow_job_templates/` details.
    1. determine the id of your workflow.
1. Go back to ssh terminal and edit ./simulate-amp-securex-POST-request
    ```yaml
     1 curl --location --request POST 'https://rhap2.mysidlabs.com/api/v2/workflow_job_templates/{{ CHANGE TO YOUR WORKFLOW ID }}/launch/' \
    ```
1. Execute the curl script:
    ```bash
    ./simulate-amp-securex-POST-request
    ```
1. In AC go to the jobs list and view your workflow in progress.

## FINIS



