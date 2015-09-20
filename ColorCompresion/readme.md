# Compressed gradient textures
## Introducción 
Muchas veces cuando queremos usar los formatos de compression DXT1 , DXT5, suelen dejar artefactos en las imagenes.

El formato dds comprime tambien el color , pasa de 8:8:8 a 5:6:5 s, dejando entre 32 ó 64 niveles en un canal. [^dxtCodeblock]

 [^dxtCodeblock]:(https://msdn.microsoft.com/en-us/library/windows/desktop/bb147243%28v=vs.85%29.aspx)
 
La idea que vamos a probar es de separar la luminancia. Esta la pondremos tanto en:
 * otra textura, 
   * ya sea dxt1 o G1, o BC4
 * en el canal alpha de una DXT5

El canal  alpha 
