# ctf-training

Binary Exploitation
https://pwnable.kr/

Luego de ingresar a la maquina huesped usando:

```fd@pwnable:~$ ssh fd@pwnable.kr -p2222```

Inspeccionando un poco los archivos que se encuentran disponibles estan los siguientes: 

![](images/fd_files.PNG)

Aqui podemos observar que se encuentra un archivo llamado flag pero no tenemos permisos debido a que el archivo flag solo tiene acceso el usuario fd_pwn y el grupo root

`uid=1002(fd) gid=1002(fd) groups=1002(fd)`

![](images/fd_c.PNG)

