# Sprint 3

## Задание 1. Анализ и планирование

### Задание 1.3. Определите домены и границы контекстов.
1. Домен: Управление Устройствами
  a. контекст: включение/выключение отопления
  b. контекст: установка желаемой температуры

2. Домен: Мониторинг Температуры
  a. контекст: получение статистической информации о температуре
  b. контекст: получение текущей температуры

### Задание 1.4. Проведите анализ архитектуры монолитного приложения. 
Минусы текущей монолитной реализации:
- только синхронные взаимодействия, потенциальное возникновение блокировок
- сложность масштабироваться под кратное увеличение количества потребителей
- развитие, добавление новой функциональности и поддержка трудозатратно, как финансово так и по ресурсам.
- на этапе развертывания система полностью становится недоступна для клиента
- сбои в какой-либо части монолита в большой вероятностью приводят к проблемам/недоступности всей системы

### Задание 1.5. Визуализируйте контекст системы. 
docs\task1\Context.puml

---

## Задание 2. Проектирование микросервисной архитектуры

### Задание 2.1. Декомпозируйте приложение на микросервисы. 
- сервис аутентификации
- сервис управления пользователями
- сервис нотификаций
- сервис технической поддержки
- сервис управления отоплением (можно оставить текущее решение на уровне монолита, паралельно переписывая на микросервис с последующим переключением на второй)
- сервис управления освещением
- сервис управления наблюдением
- сервис управления воротами
- сервис устройств и их сценариев

### Задание 2.2. Определите взаимодействия.
API Gateway, как единая точка входа, будет выступать посредником между пользователем и микросервисами.
Для избавления от минуса текущей реализации будет использоваться Kafka в связи с кратным ростом как количества пользователей, так и количества функицональности. Kafka будет обеспечивать асинхронное взаимодейтсвие как между микросервисами, так и в рамках потребностей одного микросервиса.
Каждый микросервис получает свою базу данных (PostgreSQL). 

### Задание 2.3. Визуализируйте архитектуры.
- C4 — Уровень контейнеров (Containers): docs\task2\Containers.pulm
- C4 — Уровень компонентов (Components): docs\task2\Components.pulm
- C4 — Уровень кода (Code) : docs\task2\Code.pulm

---

## Задание 3. Разработка ER-диаграммы

### Задание 3.1. Идентифицируйте сущности.
### Задание 3.2. Определите атрибуты. 
### Задание 3.3. Опишите связи. 
### Задание 3.4. Постройте ER-диаграмму.

---
---

# Базовая настройка

## Запуск minikube

[Инструкция по установке](https://minikube.sigs.k8s.io/docs/start/)

```bash
minikube start
```

## Добавление токена авторизации GitHub

[Получение токена](https://github.com/settings/tokens/new)

```bash
kubectl create secret docker-registry ghcr --docker-server=https://ghcr.io --docker-username=<github_username> --docker-password=<github_token> -n default
```

## Установка API GW kusk

[Install Kusk CLI](https://docs.kusk.io/getting-started/install-kusk-cli)

```bash
kusk cluster install
```

## Смена адреса образа в helm chart

После того как вы сделали форк репозитория и у вас в репозитории отработал GitHub Action. Вам нужно получить адрес образа <https://github.com/><github_username>/architecture-sprint-3/pkgs/container/architecture-sprint-3

Он выглядит таким образом
```ghcr.io/<github_username>/architecture-sprint-3:latest```

Замените адрес образа в файле `helm/smart-home-monolith/values.yaml` на полученный файл:

```yaml
image:
  repository: ghcr.io/<github_username>/architecture-sprint-3
  tag: latest
```

## Настройка terraform

[Установите Terraform](https://yandex.cloud/ru/docs/tutorials/infrastructure-management/terraform-quickstart#install-terraform)

Создайте файл ~/.terraformrc

```hcl
provider_installation {
  network_mirror {
    url = "https://terraform-mirror.yandexcloud.net/"
    include = ["registry.terraform.io/*/*"]
  }
  direct {
    exclude = ["registry.terraform.io/*/*"]
  }
}
```

## Применяем terraform конфигурацию

```bash
cd terraform
terraform init
terraform apply
```

## Настройка API GW

```bash
kusk deploy -i api.yaml
```

## Проверяем работоспособность

```bash
kubectl port-forward svc/kusk-gateway-envoy-fleet -n kusk-system 8080:80
curl localhost:8080/hello
```

## Delete minikube

```bash
minikube delete
```
