
 Project 13 Ansible Dynamic Assignments (Include) and Community Roles

First Step: Introducing Dynamic Assignment Into the structure

1. In the  https://github.com/<your-name>/ansible-config-mgt GitHub repository start a new branch and call it - dynamic-assignments. Then inside this folder, create a new file and name it env-vars.yml.

![create new branch](./images/entire-setupu.jpg)


2. Enure  appropriate folder structure is created.

├── dynamic-assignments
│   └── env-vars.yml
├── inventory
│   └── dev
    └── stage
    └── uat
    └── prod
└── playbooks
    └── site.yml
└── roles (optional folder)
    └──...(optional subfolders & files)
└── static-assignments
    └── common.yml

![appropriate folder structure](./images/entire-setupu.jpg)
    

3. Create a new folder - env-vars to keep each environment’s variables file. Then Create new YAML files which will use to set variables.

Your layout should now look like this.

├── dynamic-assignments
│   └── env-vars.yml
├── env-vars
    └── dev.yml
    └── stage.yml
    └── uat.yml
    └── prod.yml
├── inventory
    └── dev
    └── stage
    └── uat
    └── prod
├── playbooks
    └── site.yml
└── roles

└── static-assignments
    └── common.yml
    └── webservers.yml

![appropriate folder structure](./images/entire-setupu.jpg)
    
4. Now paste the instruction below into the env-vars.yml file.

---
- name: collate variables from env specific file, if it exists
  hosts: all
  tasks:
    - name: looping through list of available files
      include_vars: "{{ item }}"
      with_first_found:
        - files:
            - dev.yml
            - stage.yml
            - prod.yml
            - uat.yml
          paths:
            - "{{ playbook_dir }}/../env-vars"
      tags:
        - always

![edit env-vars.yml](./images/entire-setupu.jpg)


Second Step: Update site.yml with dynamic assignments

1. Update site.yml file to make use of the dynamic assignment. 

site.yml should now look like this.
---
- hosts: all
- name: Include dynamic variables 
  tasks:
  import_playbook: ../static-assignments/common.yml 
  include: ../dynamic-assignments/env-vars.yml
  tags:
    - always

-  hosts: webservers
- name: Webserver assignment
  import_playbook: ../static-assignments/webservers.yml

  ![update site.yml](./images/entire-setupu.jpg)

2. Create a role for MySQL database – it should install the MySQL package, create a database and configure users. Download Mysql Ansible Role from the community. Here, a MySQL role developed by geerlingguy is used. To preserve your your GitHub in actual state after you install a new role – make a commit and push to master your ‘ansible-config-mgt’ directory. 
- On Jenkins-Ansible server make sure that git is installed with git --version, then go to ‘ansible-config-mgt’ directory and 

- Run `git init`
- Run `git pull https://github.com/<(your-name)>/ansible-config-mgt.git`
- Run `git remote add origin https://github.com/<(your-name)>/ansible-config-mgt.git`
- Run `git branch roles-feature`
- Run `git switch roles-feature`

![create MySQL role](./images/entire-setupu.jpg)

2. Inside roles directory create your new MySQL role: Run `ansible-galaxy install geerlingguy.mysql` and rename the folder to mysql: Run `mv geerlingguy.mysql/ mysql`.

![new MysQL role](./images/entire-setupu.jpg)


3. Read README.md file, and edit roles configuration to use correct credentials for MySQL required for the tooling website.

![read README.md file](./images/entire-setupu.jpg)

4. Upload the changes into your GitHub:

- Run `git add`.
- Run `git commit -m "Commit new role files into GitHub"`
- Run `git push --set-upstream origin roles-feature`
- Now, if you are satisfied with your codes, you can create a Pull Request and merge it to main branch on GitHub.

![update changes to Github](./images/entire-setupu.jpg)

Third Step: Load Balancer roles

1. Declare a variable in defaults/main.yml file inside the Nginx and Apache roles. Name each variables enable_nginx_lb and enable_apache_lb respectively and set the values to false.

![load balancer roles](./images/entire-setupu.jpg)

- Declare another variable in both roles. Load_balancer_is_required and set its value to false as well.

![load_balancer variable](./images/entire-setupu.jpg)


2. Update both assignment and site.yml files respectively

- loadbalancers.yml file

- hosts: lb
  roles:
    - { role: nginx, when: enable_nginx_lb and load_balancer_is_required }
    - { role: apache, when: enable_apache_lb and load_balancer_is_required }

![updated loadbalancers.yml file](./images/entire-setupu.jpg)

site.yml file

     - name: Loadbalancers assignment
       hosts: lb
         - import_playbook: ../static-assignments/loadbalancers.yml
        when: load_balancer_is_required 

![updated site.yml file](./images/entire-setupu.jpg)


3. make use of `env-vars\uat.yml` file to define which loadbalancer to use in UAT environment by setting respective environmental variable to true.

![define loadbalancer](./images/entire-setupu.jpg)


4. activate load balancer, and enable nginx by setting these in the respective environment’s env-vars file.

- Run `enable_nginx_lb: true`
- Run `load_balancer_is_required: true`
- The same must work with apache LB, so you can switch it by setting respective environmental variable to true and other to false.

5. Perform a test by updating inventory for each environment and run Ansible against each environment.

![perform a test](./images/entire-setupu.jpg)



