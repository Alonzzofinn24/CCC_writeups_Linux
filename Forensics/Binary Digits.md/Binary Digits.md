# Binary Digits

## 📌 Información del reto

- **Plataforma:** PicoCTF
- **Categoría:** Forensics
- **Reto:** Binary Digits

## 📝 Enunciado

> This file doesn't look like much... just a bunch of 1s and 0s. But maybe it's not just random noise. Can you recover anything meaningful from this?

El reto proporciona un archivo binario cuyo contenido está compuesto por cadenas de `0` y `1`. El objetivo es recuperar la información original a partir de esos bits.

---

## 1. Análisis inicial

Al tratarse de un archivo `.bin`, podemos inspeccionar su contenido desde la terminal. Si observamos una gran cantidad de `0` y `1`, una posibilidad es que estos bits representen bytes.

Un byte está compuesto por **8 bits**, por lo que podemos agrupar el contenido en bloques de ocho bits y convertir cada bloque a un valor correspondiente.

> **Nota:** ASCII estándar utiliza 7 bits y define 128 caracteres. En este reto, sin embargo, trabajaremos con bytes de 8 bits, ya que la secuencia de bits termina representando los bytes de un archivo JPEG.

---

## 2. Conversión con CyberChef

Una forma sencilla de realizar la conversión es utilizar **CyberChef**.

Copiamos el contenido del archivo `.bin` en el panel de entrada y utilizamos la operación **From Binary**.

### Parámetros importantes

- **Delimiter:** indica cómo están separados los grupos de bits. En este reto debemos seleccionar la opción correspondiente al formato del contenido.
- **Byte Length:** indica cuántos bits forman cada unidad. Para trabajar con bytes utilizamos `8`.

![Configuración de From Binary](assets/image-20260915184812-nns0ikj.png)

Después de realizar la conversión, podemos obtener caracteres que no parecen texto legible. Esto no significa necesariamente que la conversión haya fallado: los bytes obtenidos pueden pertenecer a otro tipo de archivo.

Guardamos la salida obtenida como un archivo binario para conservar los bytes originales.

![Guardar la salida](assets/image-20260917104541-78f3sk3.png)

![Resultado de la conversión](assets/image-20260917103250-dbendat.png)

---

## 3. Identificación del archivo

Una vez obtenidos los bytes, utilizamos `file` para determinar qué tipo de archivo tenemos realmente.

```bash
file resultado.bin
```

Salida:

```text
resultado.bin: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1,
segment length 16, baseline, precision 8, 800x500, components 3
```

El resultado confirma que los bytes corresponden a una **imagen JPEG**, independientemente de que el archivo se llame `resultado.bin`.

---

## 4. Análisis de la cabecera JPEG

También podemos verificar manualmente los primeros bytes del archivo.

```bash
head -c 50 digits.bin
```

Por ejemplo, obtenemos:

```text
11111111110110001111111111100000000000000001000001
```

Tomamos los primeros 32 bits y los agrupamos en bloques de ocho:

```text
11111111 11011000 11111111 11100000
```

Convertimos cada byte a hexadecimal:

```text
FF D8 FF E0
```

Los primeros bytes de un archivo JPEG válido contienen una firma característica que comienza con:

```text
FF D8 FF
```

Por ello, la secuencia obtenida es consistente con un archivo JPEG.

---

## 5. Visualización de la imagen

Podemos abrir el archivo utilizando la aplicación predeterminada del sistema:

```bash
xdg-open resultado.bin
```

El sistema identifica el contenido como una imagen JPEG y la abre con el visor correspondiente.

![Imagen obtenida](assets/image-20260917105032-60p57og.png)

La imagen contiene la flag:

```text
picoCTF{h1dd3n_1n_th3_b1n4ry_cc2099d30}
```

---

# 🔄 Método alternativo: Python

También podemos realizar todo el proceso mediante Python.

El objetivo del script es:

1. Leer el archivo como texto.
2. Eliminar espacios y saltos de línea.
3. Dividir la cadena en grupos de 8 bits.
4. Convertir cada grupo de binario a un entero.
5. Convertir esos valores a bytes.
6. Guardar los bytes resultantes en un archivo binario.

```python
with open("file.bin", "r") as f:
    bits = f.read().replace(" ", "").strip()

byte_data = bytes(
    int(bits[i:i + 8], 2)
    for i in range(0, len(bits), 8)
)

with open("output.bin", "wb") as f:
    f.write(byte_data)
```

### Explicación

#### `with open(...)`

Abre el archivo y garantiza que el recurso sea cerrado correctamente al terminar el bloque.

#### `read()`

Lee todo el contenido del archivo como una cadena de texto.

#### `.replace(" ", "")`

Elimina los espacios existentes entre los bits.

Por ejemplo:

```text
001 0111 11
```

se convierte en:

```text
001011111
```

#### `.strip()`

Elimina espacios en blanco y saltos de línea que puedan encontrarse al principio o al final de la cadena.

#### `len(bits)`

Obtiene la cantidad total de bits disponibles.

#### `range(0, len(bits), 8)`

Permite recorrer la cadena avanzando de ocho en ocho posiciones.

#### `bits[i:i + 8]`

Realiza un *slicing* para obtener un grupo de ocho bits.

Por ejemplo:

```text
01011100
```

#### `int(..., 2)`

Convierte una cadena representada en base 2 a un número entero.

```python
int("01011100", 2)
```

produce:

```text
92
```

#### `bytes(...)`

Convierte los valores enteros obtenidos en una secuencia de bytes.

Una representación parcial podría verse como:

```python
b'\xff\xd8\xff\xe0\x00\x10JFIF...'
```

La `b` indica que estamos trabajando con una secuencia de bytes.

#### `open(..., "wb")`

El modo `wb` significa **Write Binary**. Permite escribir los bytes directamente en el archivo sin tratarlos como texto.

#### `f.write(byte_data)`

Escribe los bytes obtenidos en `output.bin`.

---

## 6. Ejecución en una sola línea

El mismo proceso puede realizarse directamente desde la terminal:

```bash
python3 -c 'bits=open("file.bin").read().replace(" ","").strip(); open("resultado.bin","wb").write(bytes(int(bits[i:i+8],2) for i in range(0,len(bits),8)))'
```

Posteriormente podemos verificar el resultado:

```bash
file resultado.bin
```

La salida debería identificar nuevamente el archivo como una imagen JPEG.

---

# ⚠️ Errores comunes

Un error importante es copiar el contenido mostrado por CyberChef y pegarlo directamente en un editor de texto para crear el archivo.

Los archivos binarios no deben tratarse como texto normal. Al pasar bytes arbitrarios por editores, navegadores o portapapeles pueden producirse conversiones de caracteres o interpretaciones Unicode que alteren los datos originales.

Una forma más segura es trabajar directamente con bytes mediante herramientas que soporten datos binarios.

![Ejemplo del resultado incorrecto](assets/image-20260917131131-p7bu8y3.png)

### Regla práctica en Forensics

> Cuando trabajes con archivos binarios, evita manipularlos mediante editores de texto. Utiliza herramientas y scripts que preserven los bytes originales.

---

# 🛠️ Comandos utilizados

### `file`

Analiza el contenido de un archivo para identificar su tipo real, independientemente de la extensión.

```bash
file resultado.bin
```

### `head`

Muestra el comienzo de un archivo.

En este reto utilizamos:

```bash
head -c 50 digits.bin
```

para obtener los primeros 50 bytes/caracteres.

### `xdg-open`

Abre un archivo utilizando la aplicación predeterminada del sistema.

```bash
xdg-open resultado.bin
```

### `python3`

Permite ejecutar el script utilizado para reconstruir los bytes originales.

---

# 🧠 Conceptos clave

- Un byte está compuesto por 8 bits.
- Una secuencia de bits puede representar datos distintos de texto.
- Las extensiones de archivo no determinan necesariamente su contenido real.
- Las firmas o *magic bytes* permiten identificar determinados formatos.
- Un JPEG comienza con la firma característica `FF D8 FF`.
- `file` permite identificar formatos a partir de su contenido.
- Los datos binarios deben manipularse procurando conservar los bytes originales.
- Python puede utilizarse para convertir cadenas binarias en bytes reales.

---

## 🚩 Flag

```text
picoCTF{h1dd3n_1n_th3_b1n4ry_cc2099d30}
```

‍
