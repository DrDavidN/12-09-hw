# «Управление доступом» - Дрибноход Давид Николаевич

### Задание 1. Создайте конфигурацию для подключения пользователя
1. Создайте и подпишите SSL-сертификат для подключения к кластеру.

#### Ответ:

Создаю новый namespace

![image](https://github.com/DrDavidN/12-09-hw/assets/128225763/bd212606-22df-4efb-b5fb-1ff5c241b1b7)

Создаю файл ключа

![image](https://github.com/DrDavidN/12-09-hw/assets/128225763/c19b74cf-ee3a-460c-b9bc-8acfd2451f95)

Запрос на подписание CSR ключа

![image](https://github.com/DrDavidN/12-09-hw/assets/128225763/e3500cd2-998a-4c10-9a4d-3974f6ba1e56)

Генерирую файл сертификата

![image](https://github.com/DrDavidN/12-09-hw/assets/128225763/31100bda-25e8-4c56-ba2e-9c6baec8d481)


2. Настройте конфигурационный файл kubectl для подключения.

#### Ответ:

Создаю пользователя staff и настраиваю его на использование созданного выше ключа

![image](https://github.com/DrDavidN/12-09-hw/assets/128225763/a9b57a4b-ec17-475a-a9cb-4ca33110c9f8)

Создаю новый контекст с именем staff-context и подключаю его к пользователю staff, созданному ранее

![image](https://github.com/DrDavidN/12-09-hw/assets/128225763/d7702f05-102a-48e8-a4a6-b8ac9f373daf)

3. Создайте роли и все необходимые настройки для пользователя.

#### Ответ:

Включу встроенный в Microk8s RBAC контроллер

![image](https://github.com/DrDavidN/12-09-hw/assets/128225763/1c3f674b-c179-4198-8185-7d07a8630ab1)

Применю манифест создания роли (Role) и манифест привязки роли к Namespace (RoleBinding)

![image](https://github.com/DrDavidN/12-09-hw/assets/128225763/06bd61d7-4529-4432-ac4d-7884518a99ae)


4. Предусмотрите права пользователя. Пользователь может просматривать логи подов и их конфигурацию (`kubectl logs pod <pod_id>`, `kubectl describe pod <pod_id>`).

#### Ответ:

Для проверки прав пользователя переключусь в его контекст

![image](https://github.com/DrDavidN/12-09-hw/assets/128225763/9002e3fb-95f5-4e1a-8dac-8f43661f024c)

Разверну Deployment в разрешенном для пользователя Namespace

![image](https://github.com/DrDavidN/12-09-hw/assets/128225763/fd5a3f9f-b3e2-4090-be69-2a4b5fa9e65b)

Проверю какие развернуты поды в Namespace с именем default

![image](https://github.com/DrDavidN/12-09-hw/assets/128225763/540ae0a9-5eb4-4b9f-a672-b45ea7c0a9e9)

Видно, что в Namespace с именем default нет доступа, так как он не был указан в манифесте Role.

Но если я проверю поды в Namespace с именем 12-09-hw, то список подов отобразится, т.к. на него есть разрешения в Role

![image](https://github.com/DrDavidN/12-09-hw/assets/128225763/5057f472-6425-44b4-bb80-cbf2381ac681)

Проверю логи пода

![image](https://github.com/DrDavidN/12-09-hw/assets/128225763/a919dde8-5848-468f-b938-4d3aa0c6c63b)

Проверю вывод описания пода

![image](https://github.com/DrDavidN/12-09-hw/assets/128225763/251698d8-1b83-4348-9094-535c80fd0dcd)


5. Предоставьте манифесты и скриншоты и/или вывод необходимых команд.

#### Ответ:

deployment.yaml

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-frontend
  name: nginx-only
  namespace: 12-09-hw
spec:
  selector:
    matchLabels:
      app: nginx-frontend
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx-frontend
    spec:
      containers:
        - name: nginx-app
          image: nginx:1.25.4
```

role.yaml

```YAML
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: 12-09-hw
  name: podinfo-viewer
rules:
- apiGroups: [""]
  resources: ["pods","pods/log"]
  verbs: ["get", "watch", "list"]
- apiGroups: ["extensions", "apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

rolebinding.yaml

```YAML
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pods
  namespace: 12-09-hw
subjects:
- kind: User
  name: staff
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: podinfo-viewer
  apiGroup: rbac.authorization.k8s.io
```


------
