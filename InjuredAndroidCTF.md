# Injured Android CTF

![](/images/injuredandroid/img1.png)

InjuredAndroid es una aplicacion que sirve como entrenamiento para el analisis de vulnerabilidades en Android y Bug Bounty de dispositivos mobiles, algunos de los retos que presenta son XSS, ataques mediante Deeplinks, API credentials, entre otros.

## XSS Test

Como primer reto esta **XSS Test** dentro de ella se muestra un campo para ingresar datos.

![](/images/injuredandroid/img2.png)

`<script>alert('XSS')</script>`

![](/images/injuredandroid/img3.png)
