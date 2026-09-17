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
*archivo.sh*
```bash
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

