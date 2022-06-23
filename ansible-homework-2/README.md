# Основная часть
1. Файл подготовил
```yaml
---
    elasticsearch:
      hosts:
        ubuntu:
          ansible_connection: docker
    kibana:
      hosts:
        ubuntu:
          ansible_connection: docker
```

2. Дописал ещё один play под кибану.
3. Использовал рекомендуемые модули.
4. Задание делается то, что указано.
```yaml
    - name: Install Kibana
        hosts: kibana
        tasks:
          - name: Upload tar.gz Kibana from remote URL
            get_url:
              url: "https://artifacts.elastic.co/downloads/kibana/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
              dest: "/tmp/kibana-{{ elastic_version }}-linux-x86_64.tar.gz"
              mode: 0755
              timeout: 60
              force: true
              validate_certs: false
            register: get_kibana
            until: get_kibana is succeeded
            tags: kibana
          - name: Create directrory for Kibana
            file:
              state: directory
              path: "{{ kibana_home }}"
            tags: kibana
          - name: Extract Kibana in the installation directory
            unarchive:
              copy: false
              src: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
              dest: "{{ kibana_home }}"
              extra_opts: [ --strip-components=1 ]
              creates: "{{ kibana_home }}/bin/kibana"
            tags:
              - kibana
          - name: Set environment Elastic
            template:
              src: templates/kib.sh.j2
              dest: /etc/profile.d/kib.sh
            tags: kibana
```

5. Выполнился без ошибок
6. Запустил с ключом --check
```yaml
ansible-playbook -i inventory/prod.yml site.yml --check
```
7. Запустил с ключом --diff
```yaml
ansible-playbook -i inventory/prod.yml site.yml --diff
```
8. Повторно запустил с ключом -diff, вижу, что части задания не применились повторно, то есть идемпотентны.

# Описание плейбука
1. group_vars
- java: версия JAVA и название дистрибутива;
- elastic: версия Elasticsearch и путь до домашней папки;
- kibana: версия Kibana и путь до домашней папки;

2. описание частей плейбука
- Установка JAVA\
  установка переменных, скачивание, копирование в домашнюю директорию, распаковка,
  добавление в переменные пути до JAVA;
- Установка Elasticsearch\
  установка переменных, скачивание, копирование в домашнюю директорию, распаковка,
  добавление в переменные пути до JAVA;
- Установка Kibana\
  установка переменных, скачивание, копирование в домашнюю директорию, распаковка,
  добавление в переменные пути до JAVA;

3. Тэги\
java, elastic, kibana