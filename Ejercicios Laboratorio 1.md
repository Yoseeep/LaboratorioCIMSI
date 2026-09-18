# 1. Filtrado de líneas vacías
```bash
grep -v "^ " nombre_archivo | wc -l
```
# 2. Monitorización de recursos
```bash
ps aux | sort -rk 4 | head -n 4 | tail -n 3
```
Explicación parámetros del **sort** : 
- r : orden reverso
- k numero_columna: elige la columna número numero_columna
# 3. Tratamiento de duplicados
```bash
cat nombre_archivo_origen | uniq > nombre_archivo_destino
```
Explicación uso de uniq: sirve para filtrar o eliminar las líneas repetidas de un archivo de texto o de la entrada estándar.
> No deja sobrescribir el mismo archivo utilizando la técnica que pide el enunciado. Se podría seguir concatenando con "&&":
> `cat nombre_archivo_origen | uniq > nombre_archivo_destino && cat nombre_archivo_destino > nombre_archivo_origen && rm nombre_archivo_destino`

# 4. Control de parámetros en Bash
```bash title="script_ej4.sh"
#!/bin/bash

if [ $# -ne 2 ]; then
	echo "Error de parámetros. Uso: $0 arg1 arg2"
	exit
fi

if [ $1 == $2 ]; then
	echo "IGUALES"
else
	echo "DISTINTOS"
fi
```

# 5. Script de búsqueda y borrado condicional
```bash title="script_ej5.sh"
#!/bin/bash

#Comprobar el correcto número de parámetros
if [ $# -ne 2 ]; then
        echo "Numero de argumentos incorrecto. Uso: ./script_ej5.sh extension etiqueta"
        exit
fi


coincide_extension=$(find . -iname "*.$1") #iname no distingue entre mayúsculas y minúsculas

for archivo in $coincide_extension; do
    num_apariciones=$(grep "$2" "$archivo" | wc -l)
    if [ $num_apariciones -gt 3 ]; then
        echo "Archivo encontrado: $archivo - Apariciones: $num_apariciones > 3 (Borrado)" 
        rm "$archivo"
    else
        
        echo "Archivo encontrado: $archivo - Apariciones: $num_apariciones <= 3 (No borrado)"
    fi
done
```

- Ejemplo de uso para los ejemplos que nos piden usando el script "crear_ejemplos_ej5.sh":
```bash title="crear_ejemplos_ej5.sh"
#!/bin/bash

mkdir ejemplos_ej5
echo -e "TAG TAG TAG\nTAG\nTAG" > ./ejemplos_ej5/ej5a.txt
echo -e "TAG\nHOLA\nTAG TAG TAG TAG\nHOLA TAG" > ./ejemplos_ej5/ej5b.txt
echo -e "TAG\nTAG HOLA HOLA\nTAG\nHOLA TAG" > ./ejemplos_ej5/ej5c.txt
# el -e para que se interprete los caracteres de escape

echo "Ejemplos creados en ./ejemplos_ej5/"
```
Uso:
```bash
./crear_ejemplos_ej5.sh
cd ejemplos_ej5/
../script_ej5.sh txt TAG
# Archivo encontrado: ./ej5c.txt - Apariciones: 4 > 3 (Borrado)
# Archivo encontrado: ./ej5a.txt - Apariciones: 3 <= 3 (No borrado)
# Archivo encontrado: ./ej5b.txt - Apariciones: 3 <= 3 (No borrado)
```

