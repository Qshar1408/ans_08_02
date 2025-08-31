# Домашнее задание к занятию 2 «Работа с Playbook»

### Грибанов Антон. FOPS-31

## Подготовка к выполнению

1. * Необязательно. Изучите, что такое [ClickHouse](https://www.youtube.com/watch?v=fjTNS2zkeBs) и [Vector](https://www.youtube.com/watch?v=CgEhyffisLY).
2. Создайте свой публичный репозиторий на GitHub с произвольным именем или используйте старый.
3. Скачайте [Playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.
4. Подготовьте хосты в соответствии с группами из предподготовленного playbook.

## Основная часть

1. Подготовьте свой inventory-файл `prod.yml`.

```bash
---
clickhouse:
  hosts:
    clickhouse-01:
      ansible_connection: ssh
      ansible_ssh_user: qshar
      ansible_host: 84.201.133.68
      ansible_private_key_file: ~/.ssh/digma2

vector:
  hosts:
    vector-01:
      ansible_connection: docker
      ansible_ssh_user: qshar
      ansible_host: 184.201.133.68
      ansible_private_key_file: ~/.ssh/digma2
```

2. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает [vector](https://vector.dev). Конфигурация vector должна деплоиться через template файл jinja2. От вас не требуется использовать все возможности шаблонизатора, просто вставьте стандартный конфиг в template файл. Информация по шаблонам по [ссылке](https://www.dmosk.ru/instruktions.php?object=ansible-nginx-install). не забудьте сделать handler на перезапуск vector в случае изменения конфигурации!
3. При создании tasks рекомендую использовать модули: `get_url`, `template`, `unarchive`, `file`.
4. Tasks должны: скачать дистрибутив нужной версии, выполнить распаковку в выбранную директорию, установить vector.

```bash
# Vector
- name: Install Vector
  tags: vector
  hosts: vector-01
  tasks:
    - name: Get RPM packages
      ansible.builtin.get_url:
        url: "https://packages.timber.io/vector/{{ vector_version }}/{{ item }}-{{ vector_version }}-1.x86_64.rpm"
        dest: "./{{ item }}.rpm"
      with_items: "{{ vector_packages }}"
    - name: Install RPM packages
      become: true
      ansible.builtin.yum:
        name: "{{ item }}.rpm"
        disable_gpg_check: true
      with_items: "{{ vector_packages }}"
    - name: Copy server configuration file
      become: true
      ansible.builtin.template:
        src: vector.toml
        dest: "/etc/vector/vector.toml"
        owner: "0"
        group: "0"
        mode: "0664"
    - name: Start vector service
      become: true
      ansible.builtin.service:
        name: vector
        state: restarted
```

5. Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть.

![ans_08_02](https://github.com/Qshar1408/ans_08_02/blob/main/img/ans_08_02_001.png)

5. Попробуйте запустить playbook на этом окружении с флагом `--check`.

![ans_08_02](https://github.com/Qshar1408/ans_08_02/blob/main/img/ans_08_02_003.png)

6. Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.

![ans_08_02](https://github.com/Qshar1408/ans_08_02/blob/main/img/ans_08_02_006.png)

7. Повторно запустите playbook с флагом `--diff` и убедитесь, что playbook идемпотентен.

![ans_08_02](https://github.com/Qshar1408/ans_08_02/blob/main/img/ans_08_02_005.png)

![ans_08_02](https://github.com/Qshar1408/ans_08_02/blob/main/img/ans_08_02_004.png)

8. Подготовьте README.md-файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги. Пример качественной документации ansible playbook по [ссылке](https://github.com/opensearch-project/ansible-playbook). Так же приложите скриншоты выполнения заданий №5-8

#### Ссылка на README.md (Ссылка на README.md)[![ans_08_02](https://github.com/Qshar1408/ans_08_02/blob/main/src/README.md)]

9. Готовый playbook выложите в свой репозиторий, поставьте тег `08-ansible-02-playbook` на фиксирующий коммит, в ответ предоставьте ссылку на него.

---

### Как оформить решение задания

Приложите ссылку на ваше решение в поле "Ссылка на решение" и нажмите "Отправить решение"

---
