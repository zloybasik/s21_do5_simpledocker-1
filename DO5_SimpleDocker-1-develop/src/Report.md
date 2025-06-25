# Simple Docker

## Содержание
1. [Готовый докер] (#part-1-готовый-докер)
2. [Операции с контейнером] (#part-2-операции-с-контейнером)
3. [Мини веб-сервер] (#part-3-мини-веб-сервер)
4. [Свой докер] (#part-4-cвой-докер)
5. [Dockle] (#part-5-dockle)
6. [Базовый Docker Compose] (#part-6-базовый-docker-compose)

## Part 1. Готовый докер

Скачивание образа nginx:
* sudo docker pull nginx

Проверка наличия образа:
* sudo docker images

Запуск контейнера в фоновом режиме:
* sudo docker run -d nginx

Проверка всех запущенных контейнеров:
* sudo docker ps -a

![Part_1_1](../src/Screenshots/Part_1/Part_1_1.png)

Просмотр информации о контейнере:
* sudo docker inspect fed1685077f0

![Part_1_2](../src/Screenshots/Part_1/Part_1_2.png)

Остановка контейнера:
* sudo docker stop fed1685077f0

Проверка всех запущенных контейнеров (статус контейнера Exited, т.е выключен):
* sudo docker ps -a

Проверка, что контейнер остановлен:
* sudo docker ps

![Part_1_3](../src/Screenshots/Part_1/Part_1_3.png)

Запуск контейнера с пробросом портов 80 и 443:
* docker run -d -p 80:80 -p 443:443 nginx

Перезапуск контейнера:
* sudo docker restart 1c3b3d51de61

![Part_1_4](../src/Screenshots/Part_1/Part_1_4.png)

Проверка доступности веб-ресурса с помощью curl
* curl http://localhost:80

![Part_1_5](../src/Screenshots/Part_1/Part_1_5.png)

## Part 2. Операции с контейнером  

<details>
  <summary>Разбираемся с конфигурацией nginx и отобразим статус страницы.</summary>
</p>


> Команда exec в контексте Docker используется для выполнения команды внутри запущенного контейнера.  
Чтобы прочитать конфигурационный файл nginx.conf внутри Docker контейнера через команду exec, использовал `docker exec barkopen cat /etc/nginx/nginx.conf`       
![exec barkopen cat](Screenshots/Part_2/Part_2_1.jpg)<br>nginx.conf<br>

</p>
</details>    



<details>
  <summary>Выводим статус сервера на localhost:80/status</summary>
</p>


Создаём файл `nginx.conf`, скопировав содержимое оригинального файлафайла.  

Дописываем необходимое для выполнения задачи, настроим в нем по пути /status отдачу страницы статуса сервера nginx.                                            
![Дописаный nginx.conf](Screenshots/Part_2/Part_2_2.jpg) <br>Исправленный nginx.conf<br>
 
 `location /status { ... }`: начинает блок конфигурации для обработки запросов к пути /status.
 
 `stub_status on;`: включает отдачу статуса сервера Nginx по пути /status.

Скопировал созданный файл *nginx.conf* внутрь докер-образа через команду `docker cp nginx.conf barkopen:/etc/nginx/nginx.conf`  
Проверил, что файл скопировался через `docker exec -it barkopen cat /etc/nginx/nginx.conf`                                                                       
![docker cp nginx.conf / docker exec](Screenshots/Part_2/Part_2_3.jpg) <br>Проверяем исправленный nginx.conf<br>

Перезапустил **nginx** внутри докер-образа через команду `docker exec barkopen nginx -s reload`                                                                    
![docker exec reload](Screenshots/Part_2/Part_2_3_1.jpg) <br>Пререзапуск<br>


Проверил, что по адресу `localhost:80/status` выдается страничка со статусом сервера **nginx** в тексовом браузере.                                          
![links2](Screenshots/Part_2/Part_2_4.jpg)<br>Link<br> 

Также проверил выдачу страници в браузере Chrome.                                                                                                
![http://localhost/status  /  http:// 10.31.170.216/status](Screenshots/Part_2_5.jpg )<br>Вывод по запросу<br> 


</p>
</details>  


<details>
  <summary>Сохраняем настройки с помощью export и import</summary>
</p>

Экспортировал контейнер в архив *container.tar* командой `docker export barkopen > container.tar`                                                               
![export](Screenshots/Part_2/Part_2_6.jpg)<br>Экспортируем<br>

1. Остановил контейнер командой `docker stop barkopen`, проверил статус командой `docker ps -a`   
2. Удалил образ через `docker rmi -f nginx`, проверил через `docker images`     
3. Удалил остановленный контейнер командой `docker rm barkopen`, проверил через `docker ps -a`                                                                    
![stop / rmi / rm](Screenshots/Part_2/art_2_8.jpg)<br>Дейстрия 1 - 3 <br>

Импортировал контейнер обратно командой `docker import container.tar my_image:latest`  и проверил `docker images`                                          
![import container.tar](Screenshots/Part_2/Part_2_9.jpg)<br>Импортировал контейнер<br>

Запустил импортированный контейнер командой `docker run -d -p 80:80 -p 443:443 --name barkopen my_image:latest nginx -g 'daemon off;'`
проверил через `docker ps.                                                                                                                                   
![docker run](Screenshots/Part_2/Part_2_10.jpg)<br>Запустил и проверил<br>

Проверил, что по адресу *localhost:80/status* выдается страничка со статусом сервера **nginx**.                                                             
![http://localhost/status  /  http:// 10.31.170.216/status](Screenshots/Part_2/Part_2_11.jpg)<br>Выдача по запросу<br> 


</p>
</details>



## Part 3. Мини веб-сервер

<details>
  <summary>Отчет по мини-серверу</summary>
</p>

Пишем мини сервер на C и FastCgi, который будет возвращать простейшую страничку с надписью 'Hello World!',                                                       
![Hello_World](Screenshots/Part_3/Part_3_2.jpg)<br>Вариант мини сервера на С<br>

Пишем свой nginx.conf, который будет проксировать все запросы с 81 порта на 127.0.0.1:8080                                                                  
![nginx.conf](Screenshots/Part_3/Part_3_1_1.jpg)<br>nginx.conf<br>

Заходим в контейнер командой docker exec -it [container id/container name] bash
 и в bash, обновляем репозитории, устанавливаем gcc, spawn-fcgi и libfcgi-dev                                                                                   
![docker exec](Screenshots/Part_3/Part_3_8.jpg)<br>Вход в контейнер<br>

Скомпилировали написанный мини сервер                                                                                                                           
![gcc](Screenshots/Part_3/Part_3_9.jpg)<br>Компиляция<br>

Запустили написанный мини сервер через spawn-fcgi на порту 8080
Применили изменения в настройках сервера:                                                                                                                       
![spawn-fcgi / reload](Screenshots/Part_3/Part_3_10.jpg)<br>Запускаем<br>

 Проверяем, что в браузере по localhost:81 отдается написанная страничка. 
 ![links2](Screenshots/Part_3/Part_3_11.jpg)<br>Link<br>

 ![links2 / Chrome](Screenshots/Part_3/Part_3_12.jpg)<br>Выдача по запросу<br>

 Положи файл nginx.conf по пути ./nginx/nginx.conf (это понадобится позже).                                                                                       
![Copy nginx.conf](Screenshots/Part_3/Part_3_13.jpg)<br>Последняя задача<br>

</p>
</details>




## Part 4. Свой докер  

<details>
  <summary>Создаём докер-образ для созданного сервера</summary>
</p>

Напишем Dockerfile.                                                                                                                                        

![Dockerfile](Screenshots/Part_4/Part_4_1.jpg)<br>Dockerfile<br>

Соберём написанный докер-образ через `docker build -t barkopener:part_4 .` указав имя и тег                                                                       
![docker build](Screenshots/Part_4/Part_4_2.jpg)<br>Сборка образа<br>

Проверил через `docker images`, что все собралось корректно                                                                                               
![docker images](Screenshots/Part_4/Part_4_3.jpg)<br>Проверяем наличие собранного образа<br>

</p>
</details>




<details>
  <summary>Запуск образа</summary>
</p>


Запустим собранный докер-образ с маппингом 81 порта на 80 на локальной машине и маппингом папки *./nginx* внутрь контейнера по адресу, где лежат конфигурационные файлы **nginx**'а командой `docker run -d -p 80:81 -v "$(pwd)/nginx/nginx.conf:/etc/nginx/nginx.conf" my_fastcgi_server:part_4`  
Проверил, что по *localhost:80* доступна страничка написанного мини сервера                                                                                     
![docker images](Screenshots/Part_4/Part_4_4.jpg)<br>Запуск образа<br>

![docker images](Screenshots/Part_3/Part_3_12.jpg)<br>Результат выдачи<br>

</p>
</details>


<details>
  <summary>Допиши в ./nginx/nginx.conf проксирование странички /status, 
  по которой надо отдавать статус сервера nginx.</summary>
</p>


Дописал в *./nginx/nginx.conf* проксирование странички */status*, по которой надо отдавать статус сервера.                                                       
![docker images](Screenshots/Part_4/Part_4_5.jpg)<br>Изменения в nginx.conf<br>


Перезапустил контейнер командой `docker restart <CONTAINER_NAME>`     
*после сохранения файла и перезапуска контейнера, конфигурационный файл внутри докер-образа обновился*  
Проверил, что теперь по *localhost:80/status* отдается страничка со статусом                                                                                
![links2 http:// ](Screenshots/Part_4/Part_4_6.jpg)<br>Link<br>

![docker restart / http:// ](Screenshots/Part_4_7.jpg)<br>Результат выдачи<br>
 
</p>
</details>



## Part 5. **Dockle**  

<details>
  <summary>Проверка образа на безопасность с помощью Dockle</summary>
</p>

Устанавливаем Dockle. Инструкцию смотрел тут https://habr.com/ru/companies/timeweb/articles/561378/                                                            
![Установка](Screenshots/Part_5/Part_5_1.jpg)<br>Установка<br>


Просканировал образ из предыдущего задания через `dockle barkopen:part_4`                                                                                       
![dockle](Screenshots/Part_5/Part_5_2.jpg)<br>Выявили ошибки<br>

Исправил образ так, чтобы при проверке через **dockle** не было ошибок и предупреждений                                                                         

![Dockerfile](Screenshots/Part_5/Part_5_3.jpg)<br>Обновлённый Докерфайл<br>   

Для решения ошибки **FATAL - CIS-DI-0010**  использовал команду с `dockle --ak NGINX_GPGKEY --ak NGINX_GPGKEY_PATH`,  которая позволяет подтвердить использование конкретных ключей для работы нашего nginx сервера    
Для фикса по рекомендации **INFO - CIS-DI-0005** использовал команду `export DOCKER_CONTENT_TRUST=1`  
я.   

Написал run.sh 
![run.sh](Screenshots/Part_5/Part_5_4.jpg)<br>Добавленный скрипт<br>                                                                                                          

Сборка докер-образа и проверка его работоспособности с помощью dockle                                                                                        
![dockle](Screenshots/Part_5/Part_5_10.jpg)<br>Перепроверили после изменений<br>


</p>
</details>

## Part 6. Базовый **Docker Compose**


<details>
  <summary>Отчёт</summary>
</p>

Устанавливаем docker-compose                                                                                                                                      
![Install](Screenshots/Part_6/Part_6_0.jpg)<br>Установка<br>

Парвим nginx.conf который будет проксировать все запросы с 8080 порта на 81 порт первого контейнера. 
![nginx.conf](Screenshots/Part_6/Part_6_1_1.jpg)<br>Дописали nginx.conf<br>


Пишим простой файл `docker-compose.yml`                                                                                                                       
![docker-compose](Screenshots/Part_6/Part_6_1.jpg)<br>Docker-compose.yml<br>

Поднимаем контэйнеры используя команду docker-compose up -d задав имена  nginx_proxy  и  part_6                                                                  
![docker-compose up](Screenshots/Part_6/6_2.jpg)<br>Свежие контейнерыbr>

Останавил контейнеры командой `down`                                                                                                                           
![down](Screenshots/Part_6/Part_6_3.jpg)<br>Остонока контейнеров<br>


Внёс изменения в файл `docker-compose.yml` , так как в дальнейшем будим билдить образы.                                                                          
![docker-compose.yml](Screenshots/Part_6/Part_6_4.jpg)<br>Обновлённый *.yml <br>

Все готово для использования команды `docker-compose build`                                                                                                
![docker-compose build](Screenshots/Part_6/Part_6_5_1.jpg)<br>Результат билдинга<br>

Проверяем командой `docker images` наличие образов, и запускаем  `decker-compose up -d`
`docker ps` проверяем , что всё работает                                                                                                                          
![docker images](Screenshots/Part_6/Part_6_5_2.jpg)<br>Проверяем наличие образов<br>



 
Проверил `curl` и запросом в браузере что всё работает отлично.                                                                                               
![links2 http://localhost:80](Screenshots/Part_6/Part_6_10.jpg)<br>Результат запросв в браузере<br> 



![cirl -v http://localhost:80](Screenshots/Part_6/Part_6_22.jpg)<br>Выдача `curl`<br>


 
</p>
</details>

