## inventories
[ppd-jcp-appserver1]

10.140.100.84 ansible_ssh_private_key_file=/home/userapp/.ssh/id_rsa ansible_user=userapp

[all-ppd-jcp-appserver:children]
ppd-jcp-appserver1

## hosts: sudo vim /etc/ansible/hosts

[ppd-jcp-appserver1]
10.140.100.84 ansible_ssh_pass=MARCH19 ansible_ssh_user=userapp
[all-ppd-jcp-appserver:children]
ppd-jcp-appserver1

# ppd-jcp-appserver-upgrade.yml

- hosts: all-ppd-jcp-appserver
  remote_user: userapp
  vars_files:
     - vars/vars.yml
  roles:
      - ppd-jcp-appserver-backup
      - ppd-jcp-appserver-upgrade
      - ppd-jcp-appserver-tomcat-services

      
      
	  
## Roles1: /usr/jcp/jcp-ansible/roles/ppd-jcp-appserver-backup/tasks/main.yml

- name: create /usr/jcp/backup/jcp/{{ansible_date_time.date}} directory if it doesn't exist
  file: path=/usr/jcp/backup/jcp/{{ansible_date_time.date}} state=directory owner=userapp group=userapp mode=0777

- name: check whether foresight directory exists
  stat: path=/usr/jcp/webappSanity/foresight/WEB-INF/classes 
  register: foresight_stat

- name:  backup com directory to /usr/jcp/backup/jcp/{{ansible_date_time.date}} as WEB-INF_{{ansible_date_time.time}}
  shell: cp -r /usr/jcp/webappSanity/foresight/WEB-INF/classes/com /usr/jcp/backup/jcp/{{ansible_date_time.date}}/com_$(date +"%H%M%S")
  when: foresight_stat.stat.exists

- name:  backup application.xml directory to /usr/jcp/backup/jcp/{{ansible_date_time.date}} as WEB-INF_{{ansible_date_time.time}}
  shell: cp -r /usr/jcp/webappSanity/foresight/WEB-INF/classes/applicationContext/application.xml /usr/jcp/backup/jcp/{{ansible_date_time.date}}/application.xml_$(date +"%H%M%S")
  when: foresight_stat.stat.exists
  
- name:  backup task-executors.xml directory to /usr/jcp/backup/jcp/{{ansible_date_time.date}} as WEB-INF_{{ansible_date_time.time}}
  shell: cp -r /usr/jcp/webappSanity/foresight/WEB-INF/classes/applicationContext/task-executors.xml /usr/jcp/backup/jcp/{{ansible_date_time.date}}/task-executors.xml_$(date +"%H%M%S")
  when: foresight_stat.stat.exists
  
- name:  backup persistence.xml directory to /usr/jcp/backup/jcp/{{ansible_date_time.date}} as WEB-INF_{{ansible_date_time.time}}
  shell: cp -r /usr/jcp/webappSanity/foresight/WEB-INF/classes/META-INF/persistence.xml /usr/jcp/backup/jcp/{{ansible_date_time.date}}/persistence.xml_$(date +"%H%M%S")
  when: foresight_stat.stat.exists
  
- name : list the timestamp and contents of /usr/jcp/backup/jcp/{{ ansible_date_time.date }}/ directory
  shell: ls -ld /usr/jcp/backup/jcp/{{ ansible_date_time.date }}/ && ls -ltr /usr/jcp/backup/jcp/{{ ansible_date_time.date }}/
  register: bkp_timestamp
- debug: var=bkp_timestamp.stdout.split('\n')

## Roles2: /usr/jcp/jcp-ansible/roles/ppd-jcp-appserver-upgrade/tasks/main.yml

- name: copy the artifacts com.zip from 10.140.100.66 server to Ansible Server at /usr/jcp/jcp-release/ppd-jcp-appserver/
  local_action: command rsync -az userapp@10.140.100.66:/usr/jcp/PRODBUILD/{{ ansible_date_time.date }}/com.zip /usr/jcp/jcp-release/ppd-jcp-appserver/
  run_once: true

- name: copy the artifacts application.xml from 10.140.100.66 server to Ansible Server at /usr/jcp/jcp-release/ppd-jcp-appserver/
  local_action: command rsync -az userapp@10.140.100.66:/usr/jcp/PRODBUILD/{{ ansible_date_time.date }}/application.xml /usr/jcp/jcp-release/ppd-jcp-appserver/
  run_once: true

- name: copy the artifacts persistence.xml from 10.140.100.66 server to Ansible Server at /usr/jcp/jcp-release/ppd-jcp-appserver/
  local_action: command rsync -az userapp@10.140.100.66:/usr/jcp/PRODBUILD/{{ ansible_date_time.date }}/persistence.xml /usr/jcp/jcp-release/ppd-jcp-appserver/
  run_once: true

- name: copy the artifacts task-executors.xml from 10.140.100.66 server to Ansible Server at /usr/jcp/jcp-release/ppd-jcp-appserver/
  local_action: command rsync -az userapp@10.140.100.66:/usr/jcp/PRODBUILD/{{ ansible_date_time.date }}/task-executors.xml /usr/jcp/jcp-release/ppd-jcp-appserver/
  run_once: true


- name: copy com.zip into  directory
  synchronize: src={{ JCP_BUILD_DIR }}/ppd-jcp-appserver/com.zip dest=/usr/jcp/webappSanity/foresight/WEB-INF/classes / checksum=yes


- name: delete the com directory
  file: path=/usr/jcp/webappSanity/foresight/WEB-INF/classes/com state=absent

- name : unzip the com.zip into /usr/jcp/webappSanity/foresight/WEB-INF/classes  directory
  shell: cd /usr/jcp/webappSanity/foresight/WEB-INF/classes/ && unzip -o  com.zip

- name: delete the com.zip file
  file: path=/usr/jcp/webappSanity/foresight/WEB-INF/classes/com.zip state=absent

- name: copy application.xml into /usr/jcp/webappSanity/foresight/WEB-INF/classes/applicationContext directory
  synchronize: src={{ JCP_BUILD_DIR }}/ppd-jcp-appserver/application.xml dest=/usr/jcp/webappSanity/foresight/WEB-INF/classes/applicationContext/ checksum=yes

- name: copy persistence.xml into /usr/jcp/webappSanity/foresight/WEB-INF/classes/META-INF directory
  synchronize: src={{ JCP_BUILD_DIR }}/ppd-jcp-appserver/persistence.xml dest=/usr/jcp/webappSanity/foresight/WEB-INF/classes/META-INF/ checksum=yes

- name: copy task-executors.xml into /usr/jcp/webappSanity/foresight/WEB-INF/classes/applicationContext directory
  synchronize: src={{ JCP_BUILD_DIR }}/ppd-jcp-appserver/task-executors.xml dest=/usr/jcp/webappSanity/foresight/WEB-INF/classes/applicationContext/ checksum=yes

- name : change the permissions of /usr/jcp/webappSanity/foresight/WEB-INF/classes  directory to 755 recursively
  file: dest=/usr/jcp/webappSanity/foresight/WEB-INF/classes  owner=userapp group=userapp mode=0755 recurse=yes

- name : list the timestamp and contents of /usr/jcp/webappSanity/foresight/WEB-INF/classes  directory
  shell: ls -ld /usr/jcp/webappSanity/foresight/WEB-INF/classes/ && ls -ltr /usr/jcp/webappSanity/foresight/WEB-INF/classes/
  register: com_timestamp
- debug: var=com_timestamp.stdout.split('\n')


## Roles3: /usr/jcp/jcp-ansible/roles/ppd-jcp-appserver-tomcat-services/tasks/main.yml

- name: get pid of running /usr/jcp/tomcat-sanity processes
  shell: pgrep java | xargs pwdx | grep "/usr/jcp/tomcat-sanity" | awk -F'[ :]+' '{printf "%s\n", $1}'
  register: running_tomcat_pid

- name: force kill running /usr/jcp/tomcat-sanity processes
  shell: "kill -9 {{ item }}"
  with_items: "{{ running_tomcat_pid.stdout_lines }}"

- wait_for:
    path: "/proc/{{ item }}/status"
    state: absent
  with_items: "{{ running_tomcat_pid.stdout_lines }}"

- name: start the /usr/jcp/tomcat-sanity services
  shell: nohup sh bin/catalina.sh run  > terminal-$(date +%F).log 2>&1 &
  args:
     chdir: /usr/jcp/tomcat-sanity

- name: Wait until the string "server start up date" is in the file terminal-{{ ansible_date_time.date }}.log before continuing
  wait_for:
    path: /usr/jcp/tomcat-sanity/terminal-{{ ansible_date_time.date }}.log
    search_regex: "server start up date"
    timeout: 420

- name: Check for Keyword "Exception" in the file terminal-{{ ansible_date_time.date }}.log
  shell: sed '/server start up date/q' /usr/jcp/tomcat-sanity/terminal-{{ ansible_date_time.date }}.log | grep "Exception"
  register: jcp1_exception_output
  failed_when: jcp1_exception_output.rc != 1
  changed_when: false
  args:
    warn: False

- name: sleep for 100 seconds and continue with play
  wait_for: timeout=100

- name: check the pid of running /usr/jcp/tomcat-sanity service
  shell: pgrep java | xargs pwdx | grep "/usr/jcp/tomcat-sanity" | awk -F'[ :]+' '{printf "%s\n", $1}'
  register: pidoftomcatjcp1

- debug:
    msg: "pid of tomcat-jcp1 process is {{ pidoftomcatjcp1.stdout }} {{ pidoftomcatjcp1.stderr }}"


    
# RollBack Script 

## [rootuser@jiojcpedge3 jcp-ansible]$ cat ppd-jcp-app-server-rollback.yml
## file: rollback.yml
## This is rollback playbook

- hosts: all-jcp-app
  remote_user: userapp
  vars_files:
     - vars/vars.yml
  roles:
     - ppd-jcp-app-server-rollback
     - ppd-jcp-appserver-tomcat-services
     
     

## Role1: /usr/jcp/jcp-ansible/roles/ppd-jcp-app-server-rollback/tasks/main.yml

- name: finding all date directories in /usr/jcp/backup/
  find:
    paths: "/usr/jcp/backup/"
    file_type: directory
    patterns: '^([12]\d{3}-(0[1-9]|1[0-2])-(0[1-9]|[12]\d|3[01]))$'
    use_regex: yes
  register: found_directories

- name : finding latest backup date directory in /usr/jcp/backup
  set_fact:
    latest_bkp_dir: "{{ found_directories.files | sort (attribute='mtime',reverse=true) | first }}"

- debug:
    msg: " latest backup date directory is {{  latest_bkp_dir.path }} "


- name: finding all backups in {{  latest_bkp_dir.path }}
  find:
    paths: "{{  latest_bkp_dir.path }}"
    file_type: directory
    patterns: 'WEB-INF*'
  register: found_backups

- name : getting the oldest backup from {{  latest_bkp_dir.path }} directory
  set_fact:
    oldest_bkp_dir: "{{ found_backups.files | sort (attribute='mtime') | first }}"

- debug:
    msg: " oldest backup directory is {{  oldest_bkp_dir.path }} "


- name: check whether /usr/jcp/webappSanity/foresight/WEB-INF directory exists
  stat: path=/usr/jcp/webappSanity/foresight/WEB-INF
  register: webinf_stat

- name: delete the  /usr/jcp/webappSanity/foresight/WEB-INF/classes/com directory
  file: path=/usr/jcp/webappSanity/foresight/WEB-INF/classes/com state=absent
  when: webinf_stat.stat.exists

- name: copy WEB-INF from oldest backup {{  oldest_bkp_dir.path }} into /usr/jcp/webapp/foresight direcotry
  shell: cp -ar {{ oldest_bkp_dir.path }}/. /usr/jcp/webappSanity/foresight/WEB-INF/

- name : list the timestamp and contents of /usr/jcp/webappSanity/foresight/WEB-INF directory
  shell: ls -ld /usr/jcp/webappSanity/foresight/WEB-INF/ && ls -ltr /usr/jcp/webappSanity/foresight/WEB-INF/
  register: com_timestamp
- debug: var=com_timestamp.stdout.split('\n')



## Roles2: /usr/jcp/jcp-ansible/roles/ppd-jcp-appserver-tomcat-services/tasks/main.yml

- name: get pid of running tomcat-jcp1 processes
  shell: pgrep java | xargs pwdx | grep "/usr/jcp/tomcat-jcp1/bin" | awk -F'[ :]+' '{printf "%s\n", $1}'
  register: running_tomcat1_pid

- name: force kill running tomcat-jcp1 processes
  shell: "kill -9 {{ item }}"
  with_items: "{{ running_tomcat1_pid.stdout_lines }}"

- wait_for:
    path: "/proc/{{ item }}/status"
    state: absent
  with_items: "{{ running_tomcat1_pid.stdout_lines }}"

- name: start the tomcat-jcp1 services
  shell: nohup /bin/sh catalina.sh run  > tomcat-$(date +%F).log 2>&1 &
  args:
     chdir: /usr/jcp/tomcat-jcp1/bin

- name: Wait until the string "server start up date" is in the file tomcat-{{ ansible_date_time.date }}.log before continuing
  wait_for:
    path: /usr/jcp/tomcat-jcp1/bin/tomcat-{{ ansible_date_time.date }}.log
    search_regex: "server start up date"
    timeout: 300

- name: Check for Keyword "Exception" in the file tomcat-{{ ansible_date_time.date }}.log
  shell: sed '/server start up date/q' /usr/jcp/tomcat-jcp1/bin/tomcat-{{ ansible_date_time.date }}.log | grep "Exception"
  register: jcp1_exception_output
  failed_when: jcp1_exception_output.rc != 1
  changed_when: false
  args:
    warn: False

- name: sleep for 100 seconds and continue with play
  wait_for: timeout=100

- name: check the pid of running tomcat-jcp1 service
  shell: pgrep java | xargs pwdx | grep "/usr/jcp/tomcat-jcp1/bin" | awk -F'[ :]+' '{printf "%s\n", $1}'
  register: pidoftomcatjcp1

- debug:
    msg: "pid of tomcat-jcp1 process is {{ pidoftomcatjcp1.stdout }} {{ pidoftomcatjcp1.stderr }}"



