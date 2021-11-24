```yaml
---
- name: Install Java #имя плейбука  
  hosts: all #хосты на которых выполнять таски
  tasks:
    - name: Set facts for Java 11 vars
      set_fact:
        java_home: "/opt/jdk/{{ java_jdk_version }}"
      tags: java
    - name: Upload .tar.gz file containing binaries from local storage
      copy:
        src: "{{ java_oracle_jdk_package }}"
        dest: "/tmp/jdk-{{ java_jdk_version }}.tar.gz"
      register: download_java_binaries
      until: download_java_binaries is succeeded
      tags: java
    - name: Ensure installation dir exists
      become: true
      file:
        state: directory
        path: "{{ java_home }}"
      tags: java
    - name: Extract java in the installation directory
      become: true
      unarchive:
        copy: false
        src: "/tmp/jdk-{{ java_jdk_version }}.tar.gz"
        dest: "{{ java_home }}"
        extra_opts: [--strip-components=1]
        creates: "{{ java_home }}/bin/java"
      tags:
        - java
    - name: Export environment variables
      become: true
      template:
        src: jdk.sh.j2
        dest: /etc/profile.d/jdk.sh
      tags: java
- name: Install Elasticsearch
  hosts: elk
  tasks:
    - name: Upload tar.gz Elasticsearch from remote URL
      get_url:
        url: "https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-{{ elastic_version }}-linux-x86_64.tar.gz"
        dest: "/tmp/elasticsearch-{{ elastic_version }}-linux-x86_64.tar.gz"
        mode: 0755
        timeout: 60
        force: true
        validate_certs: false
      register: get_elastic
      until: get_elastic is succeeded
      tags: elastic
    - name: Create directrory for Elasticsearch
      file:
        state: directory
        path: "{{ elastic_home }}"
      tags: elastic
    - name: Extract Elasticsearch in the installation directory
      become: true
      unarchive:
        copy: false
        src: "/tmp/elasticsearch-{{ elastic_version }}-linux-x86_64.tar.gz"
        dest: "{{ elastic_home }}"
        extra_opts: [--strip-components=1]
        creates: "{{ elastic_home }}/bin/elasticsearch"
      tags:
        - elastic
    - name: Set environment Elastic
      become: true
      template:
        src: templates/elk.sh.j2
        dest: /etc/profile.d/elk.sh
      tags: elastic
- name: Install Kibana #имя плейбука  
  hosts: elk #хосты на которых выполнять таски
  tasks:
    - name: Upload tar.gz Kibana from remote URL
      get_url: #модуль скачивает файл по url
        url: "https://artifacts.elastic.co/downloads/kibana/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
        dest: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz" #куда положить скаченный файл
        mode: 0755 #права на файл
        timeout: 60
        force: true
        validate_certs: false
      register: get_kibana #записывает результаты выполнения get_url в переменную get_kibana
      until: get_kibana is succeeded #повторять get_url пока не выполнится удачно (но не более 3х раз)
      tags: kibana #тег таска
    - name: Create directrory for Kibana
      file: #создает дирректорию
        state: directory
        path: "{{ kibana_home }}"
      tags: kibana
    - name: Extract Kibana in the installation directory
      become: true #повышение привелегий
      unarchive: #разархивирование
        copy: false
        src: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz" #архив
        dest: "{{ kibana_home }}" #куда разархивировать
        extra_opts: [--strip-components=1] #доп-опция для архиватора
        creates: "{{ kibana_home }}/bin/kibana" #проверка появился-ли файл kibana
      tags:
        - kibana
    - name: Set environment Kibana
      become: true
      template: #модуль копирует файл src в dest. при этом умеет интерполировать переменные
        src: kibana.sh.j2
        dest: /etc/profile.d/kibana.sh
      tags: kibana
```
