##### Установите ALB Ingress-контроллер

Для балансировки нагрузки и распределения трафика между приложениями Kubernetes используйте Ingress-контроллер Yandex Application Load Balancer. Он запускает балансировщик и необходимые вспомогательные ресурсы, когда пользователь создаёт ресурс Ingress в кластере Managed Service for Kubernetes.

##### 1️⃣ Создайте сервисный аккаунт, необходимый для работы Ingress-контроллера.

##### 2️⃣ Назначьте ему роли:
- `alb.editor` — для создания необходимых ресурсов.
- `vpc.publicAdmin` — для управления внешней связностью.
- `certificate-manager.certificates.downloader` — для работы с сертификатами, зарегистрированными в сервисе Yandex Certificate Manager.
- `compute.viewer` — для использования узлов кластера Managed Service for Kubernetes в целевых группах балансировщика.

##### 3️⃣ Для установки ALB Ingress Controller перейдите в консоли управления на вкладку Marketplace:
![Marketplace](https://github.com/user-attachments/assets/06c942e9-7e1e-4ae4-bf48-9cc0f4d186ea)

##### 4️⃣ Найдите в поиске ALB Ingress Controller и зайдите в это приложение:
![ALB Ingress Controller](https://github.com/user-attachments/assets/56538a55-ce64-45ad-a4f6-0713430b720b)

##### 5️⃣ Нажмите на кнопку "Перейти к установке":
![Установка](https://github.com/user-attachments/assets/fb0e06f2-4137-4aa1-a5c0-1977a364b7c3)

##### 6️⃣ Заполните параметры приложения:
![Параметры](https://github.com/user-attachments/assets/91715986-8b68-4306-8113-6c3a7a1675c1)

##### 7️⃣ Создайте новое пространство имён для установки в него ALB Ingress Controller, например `alb-ingress`.

##### 8️⃣ Создайте новый ключ для сервисного аккаунта, созданного в пункте 1.

##### 9️⃣ Установите приложение:
![Установка приложения](https://github.com/user-attachments/assets/94aa8e5c-9e57-4bf4-b107-076e0451457e)

##### 🔟 Через некоторое время (обычно менее минуты) приложение установится и изменит статус с `Pending install` на `Deployed`.

---

##### Бригадир
Теперь можно создавать Ingress в кластере, и Ingress Controller создаст для него все необходимые ресурсы в облаке.

##### Создайте Ingress
Ingress позволяет управлять внешним доступом к сервисам в кластере Kubernetes, предоставляя возможность гибкой маршрутизации HTTP- и HTTPS-трафика. В этом разделе вы создадите простой Ingress для маршрутизации HTTP-трафика.

##### 1️⃣ Создайте объект Ingress:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  namespace: nginx-with-svc
  annotations:
    ingress.alb.yc.io/subnets: <subnet_ids>
    ingress.alb.yc.io/external-ipv4-address: auto
    ingress.alb.yc.io/group-name: my-ingress-group
spec:
  rules:
    - host: my-nginx-ingress.yc-course.ru
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-service
                port:
                  number: 80
```
##### Ключевые параметры:

**Аннотации (metadata.annotations):**

- `ingress.alb.yc.io/subnets`:
  - Определяет подсети для создания ALB
  - Получить ID подсети можно командой:
    ```bash
    yc managed-kubernetes cluster list-node-groups \
        --name <cluster_name> \
        --format json | jq '.[].node_template.network_interface_specs[].subnet_ids[]'
    ```
    *(Требуется установленная утилита `jq`)*

- `ingress.alb.yc.io/external-ipv4-address`:
  - Значение `auto` - автоматическое назначение внешнего IPv4-адреса
  - Альтернативно можно указать конкретный IP

- `ingress.alb.yc.io/group-name`:
  - Произвольное имя группы Ingress (например `my-ingress-group`)
  - Группирует связанные Ingress-ресурсы

**Спецификация (spec):**

- `rules` - правила маршрутизации:
  - `host`: Доменное имя для доступа (в примере `my-nginx-ingress.yc-course.ru`)
  - `http.paths`: Настройки путей:
    - `path`: URL-путь (`/` - корневой)
    - `pathType`: Тип сопоставления (`Prefix` или `Exact`)
    - `backend`: Куда направлять трафик:
      - `service.name`: Имя сервиса (`nginx-service`)
      - `service.port.number`: Порт сервиса (`80`)

**Важные примечания:**
1. Все аннотации обязательны для работы ALB в Yandex Cloud
2. Для HTTPS нужно дополнительно настроить:
   - Сертификат в Certificate Manager
   - Аннотацию `ingress.alb.yc.io/security-groups`
3. Для доступа из интернета требуется:
   - Публичный IP (автоматически назначается при `auto`)
   - Правила firewall в облачной сети

Убедитесь, что Ingress-контроллер создан. Для этого выполните команду и проверьте, что в поле ADDRESS в выводе команды появилось значение:

##### kubectl -n nginx-with-svc get ingress 

##### $ kubectl -n nginx-with-svc get ingress
##### NAME            CLASS    HOSTS                           ADDRESS           PORTS   AGE
##### nginx-ingress   <none>   my-nginx-ingress.yc-course.ru   158.160.176.243   80      12m 
##### 3️⃣ В рамках данного урока мы не используем реальное доменное имя, добавьте в HOSTS-файл запись для созданного Ingress:
##### 158.160.176.243 my-nginx-ingress.yc-course.ru 
##### 4️⃣ Откройте в браузере страницу http://my-nginx-ingress.yc-course.ru/. Вы должны увидеть следующее содержимое:
##### Welcome to Nginx!
##### If you see this page, the Nginx web server is successfully configured and basic auth is in use.

##### Learn more about deploying and managing applications with Kubernetes in our comprehensive Kubernetes course.

##### Pod IP: nginx-deployment-59bdf598c7-9ltxc 
#####Если обновлять страницу, название Pod будет меняться, так как сервис отправляет запросы в разные Pods.
##### Вы создали самый простой Ingress для маршрутизации трафика к вашему приложению. Подробнее с различными типами сервисов и настройкой Ingress вы познакомитесь в следующей теме.
