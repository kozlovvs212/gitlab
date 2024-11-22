# Домашнее задание: GitLab

**Фамилия, имя:** Козлов Виктор  
**Занятие:** GitLab  

## Задание 1: Развертывание GitLab и настройка GitLab Runner

### Шаг 1: Развертывание GitLab с использованием Vagrant
Для развертывания GitLab локально, я использовал `Vagrantfile` из шаблона репозитория. Следуя инструкции, был развернут GitLab на виртуальной машине.

### Шаг 2: Создание проекта в GitLab
Внутри локально развернутого GitLab я создал новый проект и пустой репозиторий для выполнения задания.

### Шаг 3: Регистрация GitLab Runner
Для проекта был зарегистрирован `gitlab-runner`, который был запущен в режиме Docker. Конфигурация `gitlab-runner` выглядит следующим образом:

![GitLab Runner Screenshot](ссылка_на_скриншот_с_настройками_runner)

### Шаг 4: Скриншоты
Добавляю скриншоты настроек `docker-runner`:

- Скриншот регистрации GitLab Runner.
- Скриншот веб-интерфейса GitLab с настройками проекта.
https://docs.google.com/document/d/1Sc8Z7B_Z2mmNr7NV6u0rP2ldGddibv6dBoCC5yOM6-o/edit?usp=sharing

## Задание 2: Создание `.gitlab-ci.yml` и настройка пайплайна


2. **Создайте файл `.gitlab-ci.yml`** с нужной конфигурацией:

```yaml
stages:
  - test
  - static-analysis
  - build

# Тестирование проекта
test:
  stage: test
  image: golang:1.17
  script:
    - go test ./...
  artifacts:
    when: always
    reports:
      junit: test-results.xml
    paths:
      - test-results.xml

# Статический анализ с использованием SonarQube
static-analysis:
  stage: static-analysis
  image:
    name: sonarsource/sonar-scanner-cli
    entrypoint: [""]  # Переопределяем точку входа, если требуется
  variables:
    SONAR_HOST_URL: "http://84.201.134.248:9000"
    SONAR_LOGIN: "sqp_bf51d21b1c87ac727afeff9bf26d5cd3a3b2fa6b"  # Новый токен
  script:
    - sonar-scanner \
        -Dsonar.projectKey=netology-gitlab \
        -Dsonar.sources=. \
        -Dsonar.host.url=$SONAR_HOST_URL \
        -Dsonar.login=$SONAR_LOGIN

# Сборка проекта в Docker (для ветки master)
build:
  stage: build
  image: docker:latest
  script:
    - docker build -t myfirstproject .
  only:
    - master
  artifacts:
    when: always
    paths:
      - docker-build-output/

# Ручной запуск сборки для не-master веток
manual-build:
  stage: build
  image: docker:latest
  when: manual
  except:
    - master
  script:
    - docker build -t myfirstproject .
  artifacts:
    when: always
    paths:
      - docker-build-output/
