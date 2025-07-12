
##### после apply -f 	PVC.yaml	secret-1.yaml nginx-pod-1.yaml


<img width="1536" height="906" alt="image" src="https://github.com/user-attachments/assets/22e22618-406d-49c5-b736-2d88cf1cf6f2" />

##### Если хотим поменять index.html ( правим наш файл index.html ) 
` kubectl cp ./index.html nginx-pod:/usr/share/nginx/html/index.html ` 

##### Смотрим через контейнер 
`kubectl exec nginx-pod -- curl -s -S -u user:password http://localhost`

##### Либо же через `kubectl port-forward nginx-pod 9080:80 > /dev/null 2>&1 & `

##### удаляем либо по файлу kubectl delete pod nginx-pod ( где pod -  kind: и nginx-pod это метаданные ) 
` kind: Pod
metadata:
  name: nginx-pod
`
обязательно учесть, что все в малом регисте при удалении пишем

##### или же удаляем по файлу  kubectl delete -f <имя-файла>
