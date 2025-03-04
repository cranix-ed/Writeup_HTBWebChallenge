# Writeup_HTB_Web_Challenge
Write-up Web Challenge Hack The Box

## Table of Content
[CDNio](#CDNio)

[Pentest Notes](#Pentest-Notes)
## 	:triangular_flag_on_post: CDNio
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

## 	:triangular_flag_on_post: Pentest Notes
