# Ansible Role: LightHouse

[Ссылка на Gitlab tag v1.2 - основной playbook](https://github.com/vasi4-dev/devops-netology/tree/origin/ansible-04)

## Description

Роль устанавливает Lighthouse + Nginx на группу хостов, определённых в вашем inventory.

## Requirements

- **Ansible 2.9+**
- **инвентори с ВМ с ОС семейства ubuntu с предустановленным python3**
- **успешное подключение к хостам по ssh**

## Role Variables

Переменные для пользователей хранятся в [defaults/main.yml](defaults/main.yml) файле.
Переменные по умолчанию не предполагаемые к изменению хранятся в [vars/main.yml](vars/main.yml) файле.

Пользовательские переменные:
| Name | Default Value | Description |
| -------------------------- | ------------- | ------------------------------------------------------------------------ |
| `nginx_user_name` | ubuntu | имя пользователя на целевой машине лайтхауса - nginx |
| `nginx_ready_delay` | "5" | задержка перед проверкой готовности веб сервера в секундах |

По умолчанию переменные:
| Name | Default Value | Description |
| -------------------------- | ------------- | ------------------------------------------------------------------------ |
| `lighthouse_location_dir` | "/usr/share/nginx/html/lighthouse" | директория для конфигов lighthouse |
| `lighthouse_vcs` | "https://github.com/VKCOM/lighthouse.git" | ссылка на репозиторий с лайтхаусом |

## Dependencies

Роль не зависит ни от каких внешних ролей.
Необходимо указать пользовательские переменные, и роль можно подключать в работу вашего плейбука:

`requirements.yml:`

```yaml
---
- src: https://github.com/vasi4-dev/lighthouse-role.git
  scm: git
  version: "v1.1"
  name: lighthouse
```

Конфигурация для nginx заливается из файла `/templates/nginx.conf.j2`

Конфигурация для Lighthouse заливается из файла `/templates/lighthouse.j2`

Директория Lighthouse по умолчанию:

```
/usr/share/nginx/html/lighthouse
```

чтобы изменить её + такие параметры как:

```
{{lighthouse_vcs}} ## ссылка на репозиторий lighthouse
{{nginx_user_name}}
## имя пользователяь nginx на машине с lighthouse
{{nginx_ready_delay}}: 5
## интервал старта nginx в секундах

```

скорректируйте файл переменных для хостов lighthouse: `/vars.yml` или `/defaults/main.yml`

## Example

Пример конфигурации:

После скачки / распаковки архива необходимо убедиться в наличии директории под конфиги. Ниже пример тасков инсталяции:

```yaml
---
- name: Download & unarchive ## get_url пропущено, так как его функционал встроен в unarchive с 2.0 версии
  become: true
  ansible.builtin.unarchive:
    src: https://packages.timber.io/vector/{{ vector_version }}/vector-{{ vector_version }}-x86_64-apple-darwin.tar.gz
    dest: /usr/local/bin
    remote_src: true
  notify: Start vector service

- name: Configure Vector | ensure what directory exists
  become: true
  ansible.builtin.file:
    path: "{{ vector_config_dir }}"
    state: directory
    mode: "0755"
  tags: [config]
- name: Configure Vector | Template config
  become: true
  ansible.builtin.template:
    src: vector.yml.j2
    mode: "0755"
    dest: "{{ vector_config_dir }}/vector.yml"
  tags: [config]
- name: Configure Vector Service | Template systemd unit
```

Пример таски, отвечающей за выбор пакетного менеджера в зависимости от целевой ОС:

```yaml
- name: Package manager choosing
  ansible.builtin.include_tasks:
    file: "{{ lookup('first_found', params) }}"
    apply:
      tags: [install]
  vars:
    params:
      files:
        - "install/{{ ansible_pkg_mgr }}.yml"
  tags: [install]
```

### Example Handlers

Запуск Nginx:

```
- name: Start nginx
  become: true
  ansible.builtin.command: nginx
  changed_when: false
- name: Start nginx

```

## Tags

Так как роль только устанавливает ПО, то почти везде есть тег install.
Все теги перечислены ниже:

| Tag     | Action                                        |
| ------- | --------------------------------------------- |
| install | таски по установке пакетов                    |
| always  | таски, выполняемые всегда                     |
| config  | таски, относящиеся к настройке учётных данных |

## Troubleshooting

Lighthouse:
Если нашли баг в работе Lighthouse - то посетите [project website](https://github.com/GoogleChrome/lighthouse).
Также [репозиторий Lighthouse](https://github.com/VKCOM/lighthouse.git) из этой сборки

## License

Лицензия Nginx: [Nginx] (https://docs.nginx.com/nginx/open-source-components/)
<br>
Лицензия Google Lighthouse: [Apache v2.0 License](https://github.com/GoogleChrome/lighthouse/blob/main/LICENSE) <br>

[Google Lighthouse agreement](https://github.com/GoogleChrome/lighthouse/blob/main/LICENSE)
