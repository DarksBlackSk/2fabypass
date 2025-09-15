## Bypass 2FA

Para este ejemplo usare un laboratorio el cual es vulnerable tanto a un sql-inyeccion como a un bypass del 2fa

![image](https://github.com/user-attachments/assets/8fa08ce1-773d-4c32-a531-5433570d48e3)

me consigo un panel de login que al probar una inyeccion basica logro el acceso (bueno, aun no, como se observa cuenta con un sistema de doble autenticacion)

![image](https://github.com/user-attachments/assets/6d35c101-cb15-4191-b9b3-55417f04a06b)

aqui intentare bypassear el `2FA`, lo primero es conocer despues de cuantos intentos fallidos me saca de la sesion y despues de testear, al 4to intento fallido me saca, otro dato que necesitamos es de cuantos digitos el codigo `2FA` que es de 4 numeros, ya sabiendo estos datos abro `BurpSuite` para realizar el bypass
ubicandome en `proxy >> HTTP history` para chequear si las solicitudes se van capturando cuando inicie sesion


![image](https://github.com/user-attachments/assets/367824c2-7003-40b0-80b3-314e3be58c70)

el navegador lo configuramos para que pase a traves del proxy de `BurpSuite` (sin activar la intercepción de peticiones en burpsuite) y a continuacion vamos a:

```bash
1) acceder a http://172.17.0.2 [Solicitud GET panel de login]
2) iniciar sesion (a traves de la inyeccion sql) [Solicitud POST inicio de sesion]
3) pagina de validacion de doble Autenticacion [Solicitud GET 2FA]
4 probamos cualquier numero de 4 digitos en el 2FA [Solicitud POST 2FA]
```

como resultado debemos tener en el historico de `Burpsuite` lo siguiente:

![image](https://github.com/user-attachments/assets/b85dbd9f-6d8f-496c-83c2-15bcbae6c321)

aqui ya observamos las peticiones capturadas las cuales usaremos a continuacion:

```bash
1) vamos al apartado de configuraciones >> Session Handling
```

![image](https://github.com/user-attachments/assets/afc8fe7c-972b-431e-a7f6-eb5249054465)


ahora agregaremos una nueva regla, aqui vamos a seleccionar `Include all URL's`

![image](https://github.com/user-attachments/assets/d5edbbba-dbe5-4d74-8daa-847fdffca598)

ahora nos vamos a `rules actions`

![image](https://github.com/user-attachments/assets/69a83db7-9c3a-423a-8f8f-03787906e602)

clicamos sobre `add` y nos vamos a `run a macro`

![image](https://github.com/user-attachments/assets/ac70bfb2-3c8b-4f2d-85e9-56e914bb0267)

ahora en `selec macro` clicamos en `add` y seleccionamos las 3 primeras peticiones

![image](https://github.com/user-attachments/assets/70b72e2e-4bb5-4907-920a-b10a264959d5)

```bash
primera peticion = login (GET)
segunda peticion = login (POST)
tercero peticion = 2FA (GET)
```

nos quedaria asi:

![image](https://github.com/user-attachments/assets/bae54994-db50-48e8-bec1-043e26882038)

ahora nos vamos a historial de peticiones capturadas y enviamos la 4ta peticion (2FA POST) al `intruder`

![image](https://github.com/user-attachments/assets/13c32901-60d6-4de4-a6ef-488ec63b13ef)

ahora nos toca configurar el ataque en el `intruder` para bypassear el `2FA`

```bash
el payload lo configuramos en el valor de code (1234, en este caso)
tipo de payload = numbers
rango de numeros = desde 0 hasta 9999
formato numerico = decimal
cantidad minima de digitos = 4
cantidad maxima de digitos = 4
```

![image](https://github.com/user-attachments/assets/31985444-3a78-4c60-89d0-24805104dfdf)

ahora nos vamos a `settings >> Error Handling`

```bash
aqui vamos a establecer despues de 4 intentos fallidos que vuelva a logearse y continuar haciendo pruebas, asi como el tiempo de espera
```

![image](https://github.com/user-attachments/assets/bda548df-63aa-4d00-8a11-05e2db2fe150)

 ya configurado todo lanzamos el ataque

 ![image](https://github.com/user-attachments/assets/d9bdb3b3-018d-4dcb-8e6e-035f87c97226)

aqui esperaremos hasta conseguir el codigo de estado `302` que representa redireccion

![image](https://github.com/user-attachments/assets/faea4d60-4d8d-4643-9924-7a3654e3c429)

despues de un buen rato de espera conseguimos el codigo `302`, le damos clic derecho y seleccionamos la opcion `show response browser` y nos aparecera la siguiente ventana

![image](https://github.com/user-attachments/assets/6f6ea9aa-40a0-41ba-be3f-4d8a32f77dcd)

copiamos la URL y la pegamos en el navegador

![image](https://github.com/user-attachments/assets/4c3f5a2f-b601-4171-ba95-6cda89b1d5e3)

hemos bypasseado el `2FA` debido a una mala implementacion que la hace vulnerable a una ataque de fuerza bruta
