Установите ALB Ingress-контроллер
Для балансировки нагрузки и распределения трафика между приложениями Kubernetes используйте Ingress-контроллер Yandex Application Load Balancer. Он запускает балансировщик и необходимые вспомогательные ресурсы, когда пользователь создаёт ресурс Ingress в кластере Managed Service for Kubernetes.
1️⃣ Создайте сервисный аккаунт, необходимый для работы Ingress-контроллера.
2️⃣ Назначьте ему роли:
alb.editor — для создания необходимых ресурсов.
vpc.publicAdmin — для управления внешней связностью.
certificate-manager.certificates.downloader — для работы с сертификатами, зарегистрированными в сервисе Yandex Certificate Manager.
compute.viewer — для использования узлов кластера Managed Service for Kubernetes в целевых группах балансировщика.
3️⃣ Для установки ALB Ingress Controller перейдите в консоли управления на вкладку Marketplace:
<img width="2880" height="1520" alt="image" src="https://github.com/user-attachments/assets/06c942e9-7e1e-4ae4-bf48-9cc0f4d186ea" />

4️⃣ Найдите в поиске ALB Ingress Controller и зайдите в это приложение:
<img width="2880" height="1520" alt="image" src="https://github.com/user-attachments/assets/56538a55-ce64-45ad-a4f6-0713430b720b" />

5️⃣ Нажмите на кнопку Перейти к установке:
<img width="2880" height="1520" alt="image" src="https://github.com/user-attachments/assets/fb0e06f2-4137-4aa1-a5c0-1977a364b7c3" />

6️⃣ Заполните параметры приложения:
<img width="2880" height="1520" alt="image" src="https://github.com/user-attachments/assets/91715986-8b68-4306-8113-6c3a7a1675c1" />

7️⃣ Создайте новое пространство имён для установки в него ALB Ingress Controller, например alb-ingress.
8️⃣ Создайте новый ключ для сервисного аккаунта, созданного в пункте 1.
9️⃣ Установите приложение:
<img width="2880" height="1520" alt="image" src="https://github.com/user-attachments/assets/94aa8e5c-9e57-4bf4-b107-076e0451457e" />

🔟 Через некоторое время, обычно менее минуты, приложение установится и изменит статус с Pending install на Deployed.


Бригадир
Теперь можно создавать Ingress в кластере, и Ingress Controller создаст для него все необходимые ресурсы в облаке. 
Создайте Ingress
Ingress позволяет управлять внешним доступом к сервисам в кластере Kubernetes, предоставляя возможность гибкой маршрутизации HTTP- и HTTPS-трафика. В этом разделе вы создадите простой Ingress для маршрутизации HTTP-трафика.
1️⃣ Создайте объект Ingress:

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
Самое важное в этом манифесте:
.metadata.annotations: Аннотации для настройки Ingress Yandex Cloud Application Load Balancer (ALB):
ingress.alb.yc.io/subnets: Определяет подсети, в которых будет создан ALB. Подставьте сюда значение идентификатора подсети, в которой создана группа узлов кластера в вашем облаке. Для его определения подставьте в команду ниже название вашего кластера вместо <cluster_name> и выполните :

yc managed-kubernetes cluster list-node-groups \
    --name <cluster_name> \
    --format json | jq '.[].node_template.network_interface_specs[].subnet_ids[]' 
Для выполнения этой команды у вас должна быть установлена утилита jq.
ingress.alb.yc.io/external-ipv4-address: Определяет, что внешний IPv4-адрес будет назначен автоматически (auto).
ingress.alb.yc.io/group-name: Указывает имя группы для Ingress (my-ingress-group).
spec: Спецификация правил маршрутизации для Ingress.
rules: Набор правил маршрутизации.
host: Доменное имя, по которому будет доступен сервис. В данном случае my-nginx-ingress.yc-course.ru.
http: Определяет правила маршрутизации трафика.
paths: Список путей и их настроек.
path: Определяет путь, который будет обрабатываться. Здесь это корневой путь /.
pathType: Указывает тип пути. В данном примере это Prefix, что означает, что все запросы, начинающиеся с указанного пути, будут маршрутизироваться.
backend: Определяет backend (сервис), на который будет направляться трафик.
service:
name: Имя сервиса, к которому будет направляться трафик. Здесь это nginx-service.
port:
number: Номер порта, на который будет направляться трафик. В данном примере это порт 80.
2️⃣ Дождитесь создания Ingress и получения им публичного IP-адреса. 
⏳Это может занять несколько минут. 
Убедитесь, что Ingress-контроллер создан. Для этого выполните команду и проверьте, что в поле ADDRESS в выводе команды появилось значение:

kubectl -n nginx-with-svc get ingress 

$ kubectl -n nginx-with-svc get ingress
NAME            CLASS    HOSTS                           ADDRESS           PORTS   AGE
nginx-ingress   <none>   my-nginx-ingress.yc-course.ru   158.160.176.243   80      12m 
3️⃣ В рамках данного урока мы не используем реальное доменное имя, добавьте в HOSTS-файл запись для созданного Ingress:
158.160.176.243 my-nginx-ingress.yc-course.ru 
4️⃣ Откройте в браузере страницу http://my-nginx-ingress.yc-course.ru/. Вы должны увидеть следующее содержимое:
Welcome to Nginx!
If you see this page, the Nginx web server is successfully configured and basic auth is in use.

Learn more about deploying and managing applications with Kubernetes in our comprehensive Kubernetes course.

Pod IP: nginx-deployment-59bdf598c7-9ltxc 
Если обновлять страницу, название Pod будет меняться, так как сервис отправляет запросы в разные Pods.
Вы создали самый простой Ingress для маршрутизации трафика к вашему приложению. Подробнее с различными типами сервисов и настройкой Ingress вы познакомитесь в следующей теме.
