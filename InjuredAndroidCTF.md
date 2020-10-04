# Injured Android CTF

![](/images/injuredandroid/img1.png)

InjuredAndroid es una aplicacion que sirve como entrenamiento para el analisis de vulnerabilidades en Android y Bug Bounty de dispositivos mobiles, algunos de los retos que presenta son XSS, ataques mediante Deeplinks, API credentials, entre otros.

## XSS Test

Como primer reto esta **XSS Test** dentro de ella se muestra un campo para ingresar datos.

![](/images/injuredandroid/img2.png)

Si usamos uno de las cadenas mas comunes para probar XSS como `<script>alert('XSS')</script>` muestra el siguiente resultado.

![](/images/injuredandroid/img3.png)

El reto consiste en mostrar que es vulnerable a XSS, ademas motiva al usuario a mirar dentro del codigo fuente de la activity "DisplayPostXSS". El codigo fuente se muestra en la siguiente imagen.

![](/images/injuredandroid/img4.png)

Examinando el codigo encontramos que se declara una webview pero ademas de las configuraciones como que se ejecute el navegador de Chrome, se utiliza `setJavaScriptEnabled(true)` que es una opcion vulnerable porque habilita la ejecucion de codigo Javascript.

## Flag One Login

![](/images/injuredandroid/img5.png)

Para superar este reto debemos ingresar la flag correspondiente y enviarla para ganar acceso, en el triangulo rojo nos muestra pistas como 'La flag esta en tu nariz' y 'La Flag tambien esta en la GUI', en el codigo fuente de la clase FlagOneActivity se muestra una string en texto plano  que presumiblemente seria la flag.

![](/images/injuredandroid/img6.png)

Al ingresar esa cadena superamos el reto FlagOneLogin

![](/images/injuredandroid/img7.png)

## Flag Two: Exported Activity

Cuando una activity esta exportada como **true** esto implica que dicha activity puede ser accedida por otras aplicaciones, esto no necesariamente implica un riesgo de seguridad si ese es el proposito o si se aplican ciertos permisos para que solo tengan acceso ciertas aplicaciones. En este reto nos sugiere que hallemos la manera de pasar a traves de la aplicacion e invocar otras activities, para esto el mismo reto sugiere que se enfoque en los terminos **exported** y **activities puede ser ingresados mediante adb o drozer**.





