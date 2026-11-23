# Параметры развёртывания devops-ci-app

- Целевой узел: учебная Ubuntu VM с Docker и архитектурой Linux ARM64.
- Подключение: SSH-пользователь `devops`. Адрес VM и SSH-порт задаются
  в inventory. При подключении с хоста через VirtualBox NAT используется
  проброс `127.0.0.1:2222`.
- Публичный образ GHCR: `ghcr.io/soalko/devops-ci-app`.
- Выбранный тег исходного коммита: `9989059ef8022a65479483910861663795b329ba`.
- Ссылка для развёртывания:
  `ghcr.io/soalko/devops-ci-app:9989059ef8022a65479483910861663795b329ba`.
- Доступ на чтение: анонимный `docker pull`, пакет GHCR публичный.
- Порт контейнера: `5000/tcp`. Для локальной проверки VM используется
  `127.0.0.1:5002:5000`.
- Проверка: HTTP 200 на `/health`, тело `{"status":"ok"}`.
- Пользователь в контейнере: `app`.

Playbook загружает и запускает уже проверенный CI образ. При переходе на
другую версию обновите тег в inventory. Для привязки к точному манифесту
используйте RepoDigest в форме `ghcr.io/soalko/devops-ci-app@sha256:…`.

## 10. Ansible

Проект находится в каталоге `ansible/`. Управляющий узел — macOS с Ansible
Core 2.19. Управляемый узел — Ubuntu Server 26.04 LTS ARM64, `devops-vm-pr11` в
VirtualBox. Для этой работы используется Host-only `192.168.56.103:22`;
параметры NAT и порт 5002 выше относятся к ручной проверке из ПР10.

- `inventory/bootstrap.ini`: пользователь `student`, пароль SSH и пароль sudo
  запрашиваются интерактивно. Ansible Core 2.19 передаёт SSH-пароль через
  `SSH_ASKPASS`.
- `inventory/prod.ini`: пользователь `devops`, ключ `~/.ssh/devops_vm`,
  `sudo -n` после первичной настройки. Это inventory задано в `ansible.cfg`.
- `base` проверяет Ubuntu, устанавливает зависимости и Docker Engine из
  официального APT-репозитория Docker.
- `ssh` создаёт `devops`, устанавливает открытый ключ, настраивает sudo,
  отключает парольную аутентификацию и включает UFW для SSH и HTTP.
- `nginx` создаёт reverse proxy, включает сайт символической ссылкой,
  проверяет конфигурацию и перечитывает её через handlers.
- `app` загружает образ GHCR, запускает контейнер и проверяет его `/health`.

Версия задаётся в `group_vars/all.yml`. Текущий образ —
`ghcr.io/soalko/devops-ci-app:9989059ef8022a65479483910861663795b329ba`.
Контейнер слушает TCP 5000, публикация на ВМ — `127.0.0.1:5000:5000`.
Внешний адрес проверки — `http://192.168.56.103/health` через Nginx на
порту 80. Ожидается HTTP 200 и `{"status":"ok"}`.

Перед первым запуском сверить ключ SSH целевой ВМ и сохранить его в
`known_hosts`. В `~/.ssh/devops_vm.pub` должен находиться соответствующий
открытый ключ учётной записи автоматизации. Команды запускаются из `ansible/`.

```sh
cd ansible
ansible-galaxy collection install -r requirements.yml
ansible-playbook -i inventory/bootstrap.ini site.yml --syntax-check
ansible-playbook site.yml --syntax-check
ansible-playbook -i inventory/bootstrap.ini site.yml --ask-pass --ask-become-pass
ansible-playbook site.yml
ansible-playbook verify.yml
```

На настроенной ВМ повторный `site.yml` даёт `changed=0`. `verify.yml` читает
эффективные настройки SSH, проверяет, что контейнер работает и его локальная
проверка `/health` проходит. Для нового адреса ВМ обновите оба inventory.
