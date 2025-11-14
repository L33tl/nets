# ЛР 2 (3). Ansible + Caddy

### Всё скачаем, настроим и пинганем сервер

![ping](gaiti-2030/ping.png)

### Добавим файл на сервер и удалим

![file](gaiti-2030/file.png)
#### Видим CHANGED, значит всё сработало и файл удалился

### Создадим роль для раскатки Caddy

```roles/caddy_deploy/tasks/main.yml:```

```yml
---

- name: Install prerequisites
  ansible.builtin.apt:
    pkg:
      - debian-keyring
      - debian-archive-keyring
      - apt-transport-https
      - curl

- name: Add key for Caddy repo
  ansible.builtin.apt_key:
    url: https://dl.cloudsmith.io/public/caddy/stable/gpg.key
    state: present
    keyring: /usr/share/keyrings/caddy-stable-archive-keyring.gpg

- name: Add Caddy repo
  ansible.builtin.apt_repository:
    repo: "deb [signed-by=/usr/share/keyrings/caddy-stable-archive-keyring.gpg] https://dl.cloudsmith.io/public/caddy/stable/deb/debian any-version main"
    state: present
    filename: caddy-stable

- name: Add Caddy src repo
  ansible.builtin.apt_repository:
    repo: "deb-src [signed-by=/usr/share/keyrings/caddy-stable-archive-keyring.gpg] https://dl.cloudsmith.io/public/caddy/stable/deb/debian any-version main"
    state: present
    filename: caddy-stable

- name: Install Caddy webserver
  ansible.builtin.apt:
    name: caddy
    update_cache: true
    state: present
```

### И сам плейбук:
```yml
---

- name: Install and configure Caddy webserver
  hosts: my_servers

  roles:
    - caddy_deploy
```

### Прогоним плейбук сначала в dry-run

![dry](gaiti-2030/dry.png)

## С точносью до наличия деб пакета на хосте всё ок)

### Теперь прогоним по-настоящему

![nast](gaiti-2030/nast.png)

### Всё поднялось всё ок

![okok](gaiti-2030/okok.png)

### Берём крутой домен, заходим и видим что всё работает

![work](gaiti-2030/work.png)

### Добавим шаблон и переменные

Caddyfile.j2:
```jinja
{{ domain_name }} {
    root * /usr/share/caddy
    file_server

    log {
        output file {{ log.file }}
        format json
        level {{ log.level }}
    }
}
```

vars/main.yaml:
```yml
---

domain_name: laba-zhoski.duckdns.org

log:
  file: /var/log/caddy/access.log
  level: "INFO"
```

### Перекатим плейбук, и всё работает

![https](gaiti-2030/https.png)

## Создание и удаление файла

deploy_file.yml:
```yml
---

- name: Deploy file
  hosts: all

  tasks:
    - name: Deploy file
      ansible.builtin.copy:
        dest: $HOME/test.txt
        content: test_file_content
        mode: "644"

    - name: Get file status
      ansible.builtin.stat:
        path: $HOME/test.txt
      register: file_check_result

    - name: Print file status
      ansible.builtin.debug:
        msg: "The file exists? - '{{ file_check_result.stat.path }}"

    - name: Add line to file
      ansible.builtin.lineinfile:
        path: $HOME/test.txt
        line: "Bim bim bam bam"
        state: present

    - name: Delete file
      ansible.builtin.file:
        dest: $HOME/test.txt
        state: absent
```

### Запустим

![deply](gaiti-2030/deply.png)

file:
![filea](gaiti-2030/filea.png)

## Все работы выполнены на выданном сервере, спасибо

#
###### that's it
