# Домашнее задание к занятию 2 «Работа с Playbook»

## Подготовка к выполнению

1. * Необязательно. Изучите, что такое [ClickHouse](https://www.youtube.com/watch?v=fjTNS2zkeBs) и [Vector](https://www.youtube.com/watch?v=CgEhyffisLY).
2. Создайте свой публичный репозиторий на GitHub с произвольным именем или используйте старый.
3. Скачайте [Playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.
4. Подготовьте хосты в соответствии с группами из предподготовленного playbook.

## Основная часть

1. Подготовьте свой inventory-файл `prod.yml`.
2. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает [vector](https://vector.dev). Конфигурация vector должна деплоиться через template файл jinja2. От вас не требуется использовать все возможности шаблонизатора, просто вставьте стандартный конфиг в template файл. Информация по шаблонам по [ссылке](https://www.dmosk.ru/instruktions.php?object=ansible-nginx-install). не забудьте сделать handler на перезапуск vector в случае изменения конфигурации!
3. При создании tasks рекомендую использовать модули: `get_url`, `template`, `unarchive`, `file`.
4. Tasks должны: скачать дистрибутив нужной версии, выполнить распаковку в выбранную директорию, установить vector.

   
5. Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть.
  ![lint](https://github.com/015fanatik/ansible_hw02/blob/76e8b119d6cf5ad7f29feda888e9ef35dbac0ee5/screenshots/lint.png)

6. Попробуйте запустить playbook на этом окружении с флагом `--check`.
   ![check](https://github.com/015fanatik/ansible_hw02/blob/a75a1f0330865617162ac83daa5ce7f9d6a441f4/screenshots/check.png)
   
7. Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.
    ![diff](https://github.com/015fanatik/ansible_hw02/blob/a75a1f0330865617162ac83daa5ce7f9d6a441f4/screenshots/diff.png)
   
8. Повторно запустите playbook с флагом `--diff` и убедитесь, что playbook идемпотентен.
 ![diff2](https://github.com/015fanatik/ansible_hw02/blob/a75a1f0330865617162ac83daa5ce7f9d6a441f4/screenshots/diff2.png)
   
9. Подготовьте README.md-файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги. Пример качественной документации ansible playbook по [ссылке](https://github.com/opensearch-project/ansible-playbook). Так же приложите скриншоты выполнения заданий №5-8




10. Готовый playbook выложите в свой репозиторий, поставьте тег `08-ansible-02-playbook` на фиксирующий коммит, в ответ предоставьте ссылку на него.

---

### Как оформить решение задания

Выполненное домашнее задание пришлите в виде ссылки на .md-файл в вашем репозитории.



## **Основные компоненты**  

  # Структура каталогов
В процессе установки будем использовать 
 ```
  -/group_vars  
      /clickhouse
      /vector   
      -vector.yaml.j2  
  -/invetnory  
      -prod.yml  
  -site.yaml   
  ```

  # Назначение файлов в структуре
- *В директории group_vars мы описываем переменные и шаблоны*  
- *В файле inventory указываем группы хостов*
- *site.yml основной плейбук Ansible*
___

## **Конфигурация основного плейбука**
  
```yaml

---
- name: Install Clickhouse
  hosts: clickhouse
  handlers:
    - name: Start clickhouse service
      become: true
      ansible.builtin.service:
        name: clickhouse-server
        state: restarted
  tasks:
    - block:
        - name: Get clickhouse distrib
          ansible.builtin.get_url:
            url: "https://packages.clickhouse.com/rpm/stable/{{ item }}-{{ clickhouse_version }}.noarch.rpm"
            dest: "./{{ item }}-{{ clickhouse_version }}.rpm"
          with_items: "{{ clickhouse_packages }}"
      rescue:
        - name: Get clickhouse distrib
          ansible.builtin.get_url:
            url: "https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-{{ clickhouse_version }}.x86_64.rpm"
            dest: "./clickhouse-common-static-{{ clickhouse_version }}.rpm"

    - name: Install clickhouse packages
      become: true
      ansible.builtin.yum:
        name:
          - clickhouse-common-static-{{ clickhouse_version }}.rpm
          - clickhouse-client-{{ clickhouse_version }}.rpm
          - clickhouse-server-{{ clickhouse_version }}.rpm
      notify: Start clickhouse service

    - name: Create database
      ansible.builtin.command: "clickhouse-client -q 'create database logs;'"
      register: create_db
      failed_when: create_db.rc != 0 and create_db.rc !=82
      changed_when: create_db.rc == 0

- name: Install Vector
  hosts: vector
  become: yes
  become_user: root
  handlers:
    - name: Start vector service
      become: true
      ansible.builtin.service:
        name: vector
        state: restarted

  tasks:
    - block:
        - name: Get vector distrib
          ansible.builtin.get_url:
            url: "https://packages.timber.io/vector/{{ vector_version }}/vector-{{ vector_version }}-1.{{ ansible_architecture }}.rpm"
            dest: "/home/centos/vector-{{ vector_version }}.rpm"

    - name: Install Vector packages
      become: true
      ansible.builtin.yum:
        name:
          - vector-{{ vector_version }}.rpm

    - name: write using jinja2
      ansible.builtin.template:
         src: ./group_vars/vector.yaml.j2
         mode: 0644
         dest: /etc/vector/vector.yaml
         owner: vector
         group: vector
      notify: Start vector service



```



   

### Как оформить решение задания

Выполненное домашнее задание пришлите в виде ссылки на .md-файл в вашем репозитории.

---
