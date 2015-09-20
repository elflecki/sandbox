# Compressed gradient textures
## Introducción 
Muchas veces cuando queremos usar los formatos de compression DXT1 , DXT5, suelen dejar artefactos en las imagenes.

El formato dds comprime tambien el color , pasa de 8:8:8 a 5:6:5 s, dejando entre 32 ó 64 niveles en un canal. [^dxtCodeblock]


[^dxtCodeblock]:(https://msdn.microsoft.com/en-us/library/windows/desktop/bb147243%28v=vs.85%29.aspx)
cada bloque es de 4x4pixeles y ocupa 64 bits. tiene dos colores base de 5:6:5 y  cada pixel del bloque puede elegir entre 4 colores. ( interpolacion de los base)

 
 
La idea que vamos a probar es de separar la luminancia del croma.  la opciones serian
 * DXT5, el chroma lo pondremos en el CANAL RGB el chroma  y la luminancia el el alpha
 * usar dos texturas.
   * El crhoma en una textura dxt1, incluso probar con una textura mas pequeña
   * La luminancia. Pruebas en 
    * G1
    * DXT5 
    * BC4
 * en el canal alpha de una DXT5
 
Itentaremos extender con pruebas con otros formatos como ETC2, pvrc , BC7, ASTC.

 

El canal  alpha del DXT5 tiene un formato de compression parecido.  pero en cada bloque de 4x4 tenemos dos grises bases en 8 bits cada uno. Cada pixel puede elegir entre 8 niveles( interpolado de los base).

El BC4 es como el canal alpha del DXT5. 

Dado que en un bloque solo se pueden dar valores interpolados entre dos colores bases. Podemos imaginar como si los colores solo pudieran ir en un eje. Esto impide degradados en distintos ejes. 

Otro problema que nos enfrentaremos es el gamma correction, el spacio srgb.
Normalmente las tarjetas transforman de espacio srgb a espacio lineal, pero no entiende el espacio lineal negativo. Mas adelante explico el concepto pero tampoco puedo asegurar lo negativo con sRGB.


## Espacios de color

Haremos 2 pruebas con 2 espacio de color
Lo tipico seria Yuv , pero probaremos 2 espacios para comparar resultado.

tipos:
  * YCoCG
  * Un formato propio basado en la teoria de oposicion de colores. creo wue es pareciod al secam o al NTSC.

Por otro lado el gamma nos ayuda a tener una buena distribuciion en la textura, cierta correspondencia entre la linearidad de la percepcion y la discretizacion.

```c++
float toSrgb( float i)
{
   float a = 0.055;
   return ( 1 + a)*pow(i , 1 / 2.4) - a;
}
```

Nuestros chromas van del rango [-1,1]  para transformarlos al rango de [0,1] que guarda la textura seria
```c++
float pack( float i)
{
   return 0.5*i +0.5;
}
```
para desempaquetar
```c++
float unpack(float i)
{
    return 2*i-1;
}
```
Si no manejamos un gamma desperdiciamos precision de la textura.
Pero a este chroma si le indicamos que esta en espacio sRGB tendriamos un efecto indeseado en la tarjeta. Ya que son numeros negativos, la curva es distinta.

Entonces vamos aplicar una correcion de gama antes del pack. Esta va a tener en cuenta el signo.
```c++
float toSignedSrgb( float signedLineal)
{
     return sign(signedLineal) * toSrgb(abs(signedLineal));
}

float toSignedLineal( float toSignedSrgb)
{
    return sign(toSignedSrgb) * toLineal(abs(toSignedSrgb));
}
```


Para accelerar los calculos usaremos un gamma distinto en el chroma en lugar de la aproximacion 2.2 o sRGB, usaremos 2.0, con lo cual a la hora de desempaquetar sera una multiplicacion en vez de un pow.


```c++


float toSignedLineal( float toSignedSrgb)
{
    return sign(toSignedSrgb) * toSignedSrgb * toSignedSrgb;
}
float toSignedSrgb( float signedLineal)
{
     return sign(signedLineal) * sqrt( abs(signedLineal) );
}
```

