# Как установить Apache SUperset на Windows

Для начала вам нужно [установить WSL](https://learn.microsoft.com/ru-ru/windows/wsl/install), поскольку Docker Desktop работает с Linux.
1. После успешной установки WSL, [скачайте Docker Desktop](https://docs.docker.com/desktop/install/windows-install/), и убедитесь что во время загрузки установили опцию **Use WSL 2 instead of Hyper-V**.

2. После установки Docker Desktop, запустите его.
 
3. Нажмите на строку поиска
   ![image](https://github.com/user-attachments/assets/7d46dad4-66ca-416b-bbbe-8fcc6ead9312)
   
4. В строке поиска найдите образ superset и нажмите Pull. Дождитесь пока образ скачается.
   ![image](https://github.com/user-attachments/assets/352e20a3-c8ce-40e1-b08a-0622796c0840)
   
5. После установки образа запустите его, нажав на кнопку RUN и указав следующие параметры, где вместо "Ваш секретный ключ" вставьте строку полученную из команды(которую надо вызвать в WSL) ```openssl rand -base64 *любое число от 1 до 100*```:
   ![image](https://github.com/user-attachments/assets/0d7f1700-ffb1-4298-a0f8-20f96ea2df02)
   
6. Теперь перейдите во вкладку conteiners и откройте созданный контейнер.
   ![image](https://github.com/user-attachments/assets/5d171369-c22b-46a4-9a32-6f6b992a9f4f)
7. Перейдите во вкладку Exec
   ![image](https://github.com/user-attachments/assets/229ce047-9f89-48f0-8078-a2203f337c6e)

В этой консоли напишите следующие команды, друг за другом

```
superset db upgrade
```

Во время выполнения команды ниже укажите username и password админа суперсет, например я указал admin admin.
```
superset fab create-admin
```

```
superset load_examples
```

```
superset init
```

8. После выполнения всех команд открываем наш суперсет нажав сюда
   ![image](https://github.com/user-attachments/assets/aa4a0c88-772e-4803-8b60-ef2d002d9849)

9. Нам откроется следующее окно в браузере, залогинимся как админ.
    ![image](https://github.com/user-attachments/assets/9e4d8679-048f-429e-9879-2164edf3e37a)

10. ![image](https://github.com/user-attachments/assets/81422d67-8c23-4ddb-9856-8d77633bfb3f)
Вуаля, наш суперсет работает! Более того, так как мы закачали базу данных примеров мы можем протестировать функционал суперсета на них.

# Как загрузить нашу бд в Superset?

Итак мы уже установили WSL и Docker Desktop, поэтому инициализируем нашу бд здесь же. 
1. Скачаем образ Postgres(я скачал под тэгом 13, так как с latest у меня были какие то проблемы)
   ![image](https://github.com/user-attachments/assets/a0e8c13f-d02c-4482-96ae-ed0a66ce1b55)
2. Запустим наш образ со следующими параметрами. Имейте ввиду что в Volumes Host path мы должны указать абсолютный путь до папки с нашим csv файлом!!!!
   ![image](https://github.com/user-attachments/assets/566dc9b2-3ceb-4915-9fe0-72f0eb6b9088)
3. Запустив образ перейдем в контейнер, вкладку Exec, где перейдем в postgres командой ```psql --username=postgres --dbname=postgres```
   Теперь создадим таблицу и загрузим нашу бд следующими командами
   ```
   CREATE TABLE reqs (id varchar(32),НомерОбращения varchar(10),Дата timestamp,Состояние varchar(150),Тип varchar(200),РабочаяГруппа varchar(200),Управление varchar(300),Услуга varchar(300),СоставУслуги varchar(300),Клиент varchar(300),Год int,Просрочен varchar(200),vip varchar(200),Хэштег varchar(300),ДатаВыполнения timestamp,НормативнаяДатаЗакрытия timestamp,Месяц varchar(100),ИмяМесяца varchar(100),КатегорияСостояний varchar(200),ДатаВремя timestamp,КодЗакрытия varchar(300),ЗакрытПоШаблону varchar(300),НаОснованииМассового varchar(300),ДатаЗагрузкиДанных timestamp,ДатаЗагрузкиДанных_last timestamp,kol_sla varchar(300),long_sla varchar(300),diff_days integer,АвтообработкаПрав varchar(300),ПоследнийНарядЗакрыт varchar(200));
   ```
   ```
   COPY reqs FROM '/dbcsv/заявки_безтемы.csv' DELIMITER '*' CSV HEADER;
   ```
   Выйдем из postgres командой ```\q```
4. Теперь давайте подключим бд в Superset, для этого перейдем в Database Connection
   ![image](https://github.com/user-attachments/assets/c957627f-adff-4dfb-8214-ea8d3edd7935)
   ![image](https://github.com/user-attachments/assets/33a9e765-07b7-4877-8900-6b9be6bb959c)
Полключим бд используя следующий настройки (в пароль пишем 123456 или что мы там указали в POSTGRES_PASSWORD, остальное не меняем)
![image](https://github.com/user-attachments/assets/0b97c497-f5ae-4430-a819-e5800b190396)
Теперь вы можете создать датасет на основе таблицы в этой бд и делать графики по ней!
![image](https://github.com/user-attachments/assets/682c5b9c-72bf-4cce-838d-05b4b2a7d220)

