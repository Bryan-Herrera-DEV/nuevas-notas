Lo principal de la arquitectura hexagonal no es solo los nombres de las carpetas, es sobre todo las reglas de dependencias y que van de afuera hacia adentro

![image](asset:13fbb5e43cf549f4bb0d8b5dcc143a1f)

Por que sucede y por que esto es bueno?

Porque las reglas de dependencia nos permiten cambiar lo de fuera sin que lo de dentro se entere y en viceversa

esto beneficia sobre todo los terminos de cambiabilidad


![image](asset:f0a2506c7684461b927d803cdd271a50)

Infra
Todo lo que va atacar al IO, todo lo que pueda llegar a ser logica pura como tocar servicios externos, base de datos, un redis, librerias,
o, lo que cambie un estado externo

Las interfaces van a ser la  