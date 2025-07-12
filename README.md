### по тегам ( для 1 все файлы только с -1.yaml )
##### после apply -f 	nginx-config-1.yaml	nginx-pod-1.yaml

##### пробросим порт `kubectl port-forward nginx-pod 9080:80`

<img width="1268" height="318" alt="image" src="https://github.com/user-attachments/assets/8f5c4712-9589-46fd-8f40-26ff38bfe459" />


<img width="2014" height="884" alt="image" src="https://github.com/user-attachments/assets/0d626784-97ad-458c-83cd-e60c998cd51a" />


##### удаляем либо по файлу kubectl delete pod nginx-pod ( где pod -  kind: и nginx-pod это метаданные ) 
` kind: Pod
metadata:
  name: nginx-pod
`
обязательно учесть, что все в малом регисте

##### или же удаляем по файлу  kubectl delete -f <имя-файла>
