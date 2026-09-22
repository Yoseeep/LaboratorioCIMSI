# 1. Filtrado de líneas vacías
```bash
grep -v "^ " nombre_archivo | wc -l
```
# 2. Monitorización de recursos
```bash
ps aux | tail -n +2 | sort -nrk 4 | head -n 3
```
Explicación parámetros del **sort** : 
- r : orden reverso
- k numero_columna: elige la columna número numero_columna
- n: numéricamente
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
        echo "Numero de argumentos incorrecto. Uso: $0 extension etiqueta"
        exit
fi

for archivo in *."$1"; do
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
# el -e para que se interpreten los caracteres de escape

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

# 6. Script de monitorización de recursos
```bash title="script_ej6.sh"
#!/bin/bash

# 1. Validación de parámetros con código de error (exit 1)
if [[ $# -ne 2 ]]; then
        echo "Error: Número de argumentos incorrecto. Uso: $0 <numero> <cpu|mem>"
        exit
fi

n="$1"
p="$2"

# 3. Mapeo de columnas
case "$p" in
        cpu|CPU)
                orden="3"
                ;;
        mem|MEM)
                orden="4"
                ;;
        *)
                echo "Error: El segundo argumento debe ser mem o cpu."
                exit 1
                ;;
esac

echo "Top $n procesos usando más $p:"

ps aux | tail -n +2 | sort -nrk "$orden" | head -n "$n"
```

# 7. Script de renombrado de extensiones
```bash title="script_ej7.sh"
#!/bin/bash

#Comprobar el correcto número de parámetros
if [ $# -ne 2 ]; then
	echo "Numero de argumentos incorrecto. Uso: ./script_ej7.sh extension_original extension_nueva"
	exit
fi

# Elimina el punto de la extensión si se proporciona
extension_original=${1#.}
extension_nueva=${2#.}

for archivo in *.$extension_original; do # recorre todos los archivos del directorio actual con la extensión original
	if [ -f "$archivo" ]; then # si es tipo archivo
		echo "Renombrando $archivo a ${archivo%.$extension_original}.$extension_nueva"
		mv -n "$archivo" "${archivo%.$extension_original}.$extension_nueva"
	fi
done
```
- **mv -n** para que en el que exista un archivo igual al renombrado aborte el  renombrado para no sobreescribirlo.
#  8. Script de renombrado recursivo
```bash title="script_ej8.sh"
#!/bin/bash

#Comprobar el correcto número de parámetros
if [ $# -ne 2 ]; then
	echo "Numero de argumentos incorrecto. Uso: ./script_ej8.sh extension_original extension_nueva"
	exit
fi

# Eliminar el punto de la extensión si se proporciona
extension_original=${1#.}
extension_nueva=${2#.}   

find . -type f -name "*.$extension_original" | while read -r archivo; do
	if [ -f "$archivo" ]; then
		echo "Renombrando $archivo a ${archivo%.$extension_original}.$extension_nueva"
		mv -n "$archivo" "${archivo%.$extension_original}.$extension_nueva"
	fi
done
```
- **read -r** (raw) para que con considere caracteres de escape(\\) como simple texto, de forma que mantiene los espacios (\ ) como parte de la palabra en lugar de separarla.

# 9. Script de detención de duplicados en el árbol de directorios
```bash title="script_ej9.sh"
#!/bin/bash

#Compruebo que solo recibe 1 parámetro

if [ "$#" -ne 1 ]; then
        echo "Uso: $0 <directorio>"
        exit
fi

#Almaceno el directorio recibido en una variable

DIR="$1"

#Compruebo que es un directorio válido, si no existe falla

if [ ! -d "$DIR" ]; then
        echo "Error: '$DIR' no es un directorio válido"
        exit
fi

#Creo archivos temporales para guardar los tamaños, rutas
#y los hash de los archivos

tmp_files=$(mktemp)
tmp_hashes=$(mktemp)

#Cuando el script finalice, se borrarán los archivos temporales

trap 'rm -f "$tmp_files" "$tmp_hashes"' EXIT

#Guardo el tamaño y la ruta de los archivos .txt que voy encontrando

find "$DIR" -type f -name "*.txt" -printf '%s %p\n' > "$tmp_files"

#Leo todo el archivo con awk
#Creo un diccionario donde las claves son los tamaños y
#los valores son las rutas de los archivos con ese tamaño exacto
#Después, imprime todos los resultados que coinciden de tamaño eliminando
#con sed los espacios en blanco
#Ahora comenzamos a recorrer las rutas que tienen el mismo tamaño y apunta
#su hash y su ruta

awk '{ count[$1]++; files[$1] = files[$1] "\n" $2 } END { for (size in count) { if (count[size] > 1) print files[size] } }' < "$tmp_files" | sed '/^$/d' | while IFS= read -r file; do
    if [ -f "$file" ]; then
        sha256sum "$file" >> "$tmp_hashes"
    fi
done

#Ordenamos la lista de hashes e imprimimos solo los que están repetidos
#Luego apuntamos todas las rutas de los archivos duplicados
#Luego revisamos los hash de manera combinatoria con un doble for
#para ir impriendo los archivos que son iguales

awk '{print $1}' "$tmp_hashes" | sort | uniq -d | while read -r hash; do
    matching_files=($(grep "^$hash" "$tmp_hashes" | awk '{print $2}'))
    num_files=${#matching_files[@]}
    for (( i=0; i<num_files; i++ )); do
        for (( j=i+1; j<num_files; j++ )); do
            file1="${matching_files[i]}"
            file2="${matching_files[j]}"
            if cmp -s "$file1" "$file2"; then
                echo "DUPLICADO ENCONTRADO:"
                echo "  Archivo 1: $file1"
                echo "  Archivo 2: $file2"
                echo "--------------------------------------------------"
            fi
        done
    done
done
```

# 10. Script de administración del sistema
```bash title="script_ej10.sh"
#!/bin/bash

#Comprobar el correcto número de parámetros
if [ $# -ne 3 ]; then
	echo "Numero de argumentos incorrecto.
Uso 1: $0 backup <origen> <destino>
Uso 2: $0 audit <directorio> <tamaño_MB>
Uso 3: $0 logs <archivo_log> <palabra_clave>"
	exit
fi

case "$1" in
	backup)
		if [ ! -d "$2" ]; then
			echo "El directorio de origen no existe"
			exit
		fi
		if [ ! -d "$3" ]; then
			echo "El directorio de destino no existe"
			exit
		fi
		tar -czf "$3/backup_$(date +%Y-%m-%d_%H%M).tar.gz" "$2"
		# f: nombre del archivo del/al que se (des)empaqueta
		# c: crea el archivo dado en la opción f (uso: empaquetar)
		# z: usa compresión (uso: empaquetar y desempaquetar)
		echo "Backup realizado correctamente"
		;;
		
	audit)
		if [ ! -d "$2" ]; then
			echo "El directorio no existe"
			exit
		fi
		touch informe_espacio.txt
		> informe_espacio.txt # vacía el archivo
		resultado=$(find "$2" -type f -size +"$3"M -exec ls -l --block-size=M {} \; | sort -nrk 5)
		# sort -n: ordena numéricamente
		# sort -r: ordena de mayor a menor
		# sort -k 5: ordena por la quinta columna (tamaño)
		echo "$resultado" | while read -r p1 p2 p3 p4 peso p6 p7 p8 nombre; do
			linea="$nombre: $peso"
			echo -e "$linea" >> informe_espacio.txt
			# echo -e para que interprete las secuencias de escape 
		done
		;;

	logs)
		if [ ! -f "$2" ]; then
			echo "El archivo de log no existe"
			exit
		fi
		grep "$3" "$2" | # Filtra las líneas que contienen la palabra clave ($3) en el archivo ($2).
		grep -Eo '\b([0-1][0-9]|2[0-3]):[0-5][0-9]:[0-5][0-9]\b' | # -E: Activa Regex extendida. -o: Extrae solo el texto coincidente.
		cut -d: -f1 | # -d: Define ':' como delimitador. -f1: Extrae la primera columna (HH).
		sort | # Ordena estructuralmente los datos (paso previo obligatorio para 'uniq').
		uniq -c | # -c: (Count) Condensa líneas idénticas y añade recuento total.
		sort -nr | # -n: Evalúa numéricamente. -r: Invierte el orden (de mayor a menor).
		while read -r incidencias hora; do # -r: Inhabilita interpretación de barras invertidas.
			echo "Hora: ${hora}:00 -> Incidencias: $incidencias"
		done
		;;

	*)
		echo "Opción incorrecta.
Uso 1: ./script_ej8.sh backup <origen> <destino>
Uso 2: ./script_ej8.sh audit <directorio> <tamaño_MB>
Uso 3: ./script_ej8.sh logs <archivo_log> <palabra_clave>"
		exit
esac
```
