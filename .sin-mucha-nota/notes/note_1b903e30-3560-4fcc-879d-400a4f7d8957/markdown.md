Lo principal de la arquitectura hexagonal no es solo los nombres de las carpetas, es sobre todo las reglas de dependencias y que van de afuera hacia adentro

![image](asset:13fbb5e43cf549f4bb0d8b5dcc143a1f)

Por que sucede y por que esto es bueno?

Porque las reglas de dependencia nos permiten cambiar lo de fuera sin que lo de dentro se entere y en viceversa

esto beneficia sobre todo los terminos de cambiabilidad


![image](asset:f0a2506c7684461b927d803cdd271a50)

Infra
Todo lo que va atacar al IO, todo lo que pueda llegar a ser logica pura como tocar servicios externos, base de datos, un redis, librerias,
o, lo que cambie un estado externo

Las interfaces van a ser la la capa para poder comunicar con aplicacion

Aplicacion 
Son los casos de uso comunes: crear un post, publicar un video, hacer una transferencia, etcT
Todo tiene que ir a travez de las implementaciones

![image](asset:e57ab57291f448e884a3358192c5e0a5)

![image](asset:e73bd8d4e9024783a4b3e0eb82d6da28)

