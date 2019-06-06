---
# tasks file for launch-make
- name: Configure and make
  shell: cd /home/{{ansible_ssh_user}}/postgres-REL_11_3 && ./configure --prefix=/opt/pg113 --with-openssl  --with-selinux --with-python
  register:  config_return_value
- name:  make main program
  shell: cd /home/{{ansible_ssh_user}}/postgres-REL_11_3 && make
  when: config_return_value.rc == 0
- name:  make contrib programs
  shell: cd /home/{{ansible_ssh_user}}/postgres-REL_11_3/contrib && make
- name:  make install main program
  become: true
  shell: cd /home/{{ansible_ssh_user}}/postgres-REL_11_3 && make install
- name:  make install contrib programs
  become: true
  shell: cd /home/{{ansible_ssh_user}}/postgres-REL_11_3/contrib && make install
- name: create password for postgres user
  command: openssl rand -hex 7
  register: postgres_password
- name: Ensure group postgres exists
  become: true
  group:
    name:  postgres
    state: present
- name: create user postgres
  become: true
  user:
    name: postgres
    shell: /bin/bash
    groups: postgres
    append: yes
    password: "{{ postgres_password.stdout }}"
- debug: msg=" New postgres password is {{ postgres_password.stdout }}"
