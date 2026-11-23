# devops-ci-app

Учебное Flask-приложение для практической работы № 10.

CI: Pylint → pytest → Docker build → smoke-test → GHCR.

Pull Request запускает проверки, сборку и smoke-test. Публикация тегов SHA и
latest выполняется при push в main. Job image использует GitHub-hosted ARM64
runner для совместимости с учебной VM.

- Health endpoint: `/health`.
- Образ: `ghcr.io/soalko/devops-ci-app`.
- Workflow: `.github/workflows/ci.yml`.

Git-тег `v1.1.0` публикует Docker-тег `1.1.0`, сохраняя `latest` за веткой
`main`. Digest каждой публикации записывается в журнал Actions.

Параметры учебной ВМ и образа приведены в [docs/server-setup.md](docs/server-setup.md).
