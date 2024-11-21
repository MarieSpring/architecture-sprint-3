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
- сервис аутентификации (AuthService)
- сервис управления пользователями (UserService)
- сервис телеметрии (TelemetryService)
- сервис управления устройствами (воротам, освещением, наблюдением, отоплением) (DeviceService)
- сервис управления сценариями (ScenarioService)
- сервис управления домами (HouseManagementService)

### Задание 2.2. Определите взаимодействия.
Пользователь использует мобильное приложение. 
Которое в свою очередь использует API Gateway как едиую точку входа, которая будет выступать посредником между приложением и микросервисами.
Для избавления от минуса текущей реализации будет использоваться Kafka в связи с кратным ростом как количества пользователей, так и количества функицональности. Kafka будет обеспечивать асинхронное взаимодейтсвие как между микросервисами, так и в рамках потребностей одного микросервиса.
Каждый микросервис получает свою базу данных (PostgreSQL). 

### Задание 2.3. Визуализируйте архитектуры.
- C4 — Уровень контейнеров (Containers): docs\task2\Containers.puml
- C4 — Уровень компонентов (Components): docs\task2\Components.puml
- C4 — Уровень кода (Code) : docs\task2\Code.puml

---

## Задание 3. Разработка ER-диаграммы

### Задание 3.1. Идентифицируйте сущности.
- модуль (Module)
- пользователь (User)
- дом (House)
- устройство (Device)
- состояние устройства (DeviceStatus)
- телеметрия (TelemetryData)

### Задание 3.2. Определите атрибуты. 
1. пользователь (User)
   - `id` — уникальный идентификатор пользователя.
   - `name` — имя пользователя.
  - `password` — пароль пользователя в зашифрованном виде.
   - `email` — электронная почта.
   - `phone` — телефон.

2. дом (House)
   - `id` — уникальный идентификатор дома.
   - `address` — адрес дома. 

3. устройство (Device)
   - `id` — уникальный идентификатор устройства.
   - `name` — наименование устройства.
   - `serialNumber` — серийный номер устройства.
   - `type` — тип устройства — Enum, выбор из перечня (ворота, освещение, наблюдение, отопление и т.д).
   - `houseId` — идентификатор дома, к которому принадлежит устройство (внешний ключ к таблице `House`).
   
4. состояние устройства (DeviceStatus)
    - `id` — уникальный идентификатор.
    - `deviceId` — уникальный идентификатор устройства (внешний ключ к таблице `Device`).
    - `status` — статус устройства.
    - `createdAt` — время фиксации статуса устройства.    

5. телеметрия (TelemetryData)
   - `id` — уникальный идентификатор значения показателя датчика.
   - `deviceId` — идентификатор устройства (внешний ключ к таблице `Device`).   
   - `value` — данные показателя.
   - `createdAt` — время фиксации показателя.

2. модуль (Module)
   - `id` — уникальный идентификатор модуля.
   - `name` — наименование модуля. 
   - `deviceId` — уникальный идентификатор устройства (внешний ключ к таблице `Device`).

### Задание 3.3. Опишите связи. 
- Пользователь — Дом (1 ко многим): один пользователь может иметь доступ к нескольким домам, но каждый дом связан только с одним пользователем.
- Дом — Устройство (1 ко многим): один дом может содержать несколько устройств, и каждое устройство принадлежит только одному дому.
- Устройство — Телеметрия (1 ко многим): одно устройство может генерировать множество записей телеметрии.
- Устройство — Состояние устройства (1 ко многим): у одного устройство может быть множество записей о состоянии в разное время.

### Задание 3.4. Постройте ER-диаграмму.
ER-диаграмма: docs\task3\ER.puml
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
