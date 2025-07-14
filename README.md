

##### удаляем либо по файлу kubectl delete pod nginx-pod ( где pod -  kind: и nginx-pod это метаданные ) 
` kind: Pod
metadata:
  name: nginx-pod
`
обязательно учесть, что все в малом регисте при удалении пишем

##### или же удаляем по файлу  kubectl delete -f <имя-файла>
