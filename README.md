# Equipo Skibidi Corp. 

## Integrantes:

- [323067360] - Miranda Valdez Alan Hjaziel
- [426058995] - Cristobal Velazco Gustavi Dainzu
- [426058555] - Hernández González Jocelyn

---

# Tema 2: Bloom filter

# Objetivo: 

Mostrar como un arreglo de bits y varias funciones hash responden "¿esta?" con muy poco espacio, a cambio de aceptar falsos positivos

---

# ¿Que es Bloom-filter?
- Es un filtro donde se deternima si el dato que buscamos existe o no, es decir es un estructura de datos donde se hace uso de hashing para poder agregar y determinar si existe o no el dato que estamos bsucando.}
- Hace uso de una matriz de bits donde estos tienen que estar inicializados en 0, para poder guardar la informacion dentro de esta.
- Se hace uso de hashing para poder almacenar solo los valores, ya que no se busca almacenar la propia clave real, Siendo que se requieren menos de 10 bits por clave.
- Optimizando el uso de memoria y no depende del tamaño de las claves individuales.
- Una desventaja que tiene el uso de Bloom-filter es que existen lso falsos positivos, siendo que el filtro podria decir que el valor del usuario ya esta en uso cuando en verdad no es asi.

---

# Contexto: 
El filtro de Bloom fue inventado en 1970 por el informatico estadounidense Burton Howard Bloom, mientras trabajaba en la empresa Computer Usage Company en Massachusetts. Lo dio a conocer en julio de ese año mediante el articulo titulado "Space/Time Trade-offs in Hash Coding with Allowable Errors" en la revista Communications of the ACM.

Donde Bloom propuso abandonar la exactitud absoluta y aceptar un pequeño margen de error controlado.

---

# Funcionamiento del filtro Bloom
- Un filtro Bloom vacio es una matriz que contiene una matriz de m cantidad de bits:
```
                    ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
                Bit │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │
                    └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
                Idx   1   2   3   4   5   6   7   8   9  10
```
Se necesitan k funciones de hash para que se pueda calcular los hashes de una entrada. Donde cuando se añade un elemento al filtro se activan en los indices h1(x), h2(x) ... hk(x) donde en los indeces se calcula por la funcion hash.

## Ejemplo

Se quiere introducir la palabra "geeks" donde se hace uso de las funciones hash con un array de longitud 10.
Primero se debe calcular los hashes:

```text
h1(“geeks”) % 10 = 1
h2(“geeks”) % 10 = 4
h3(“geeks”) % 10 = 7
```

Ahora con estas salidas los bits entran en los indices 1, 4, 7 dentro de la matriz y se cambian a 

```
                      ┌──────── geeks ────────┐
                      │           │           │
                      ▼           ▼           ▼
                    ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
                Bit │ 1 │ 0 │ 0 │ 1 │ 0 │ 0 │ 1 │ 0 │ 0 │ 0 │
                    └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
                Idx   1   2   3   4   5   6   7   8   9  10
```
            
Ahora de nuevo se queire introducir un dato, en este caso "nerd", donde igualmente se calculara mediante hashes

```text
h1(“nerd”) % 10 = 3
h2(“nerd”) % 10 = 5
h3(“nerd”) % 10 = 4
```

Donde ahora los bits a modificar seran 3, 5, 4 dentro de la matriz
```
                              ┌ nerd ─┐
                              │   │   │
                              ▼   ▼   ▼
                    ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
                Bit │ 1 │ 0 │ 1 │ 1 │ 1 │ 0 │ 1 │ 0 │ 0 │ 0 │
                    └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
                Idx   1   2   3   4   5   6   7   8   9  10


```

# Propiedades de Bloom-filter
- A la diferencia de una tabla hash el filtro Bloom de un tamaño fijo puede representar un conjunto con un numero arbitrariamente grande de elementos.
- Al momento de agregar un elemento esta nunca falla, pero la tasa de los falsos positivos va a aumetar a medidia que se llegan a agregar muchos mas elementos a la matriz de bits, donde si los bits del filtro se convierten en 1, el resultado arrogado es positivo
- No pueden darte un falso negativo, ya que no te dicen si un valor del usuario no existe cuando en realidad si existe 
- No es posible eliminar los elementos dentro del filtro ya que al eliminar un elemento se eliminan los bits en los indeces por las funciones hash k, donde se podrian eliminar tambien otros elementos.

---

# ¿Por que son estos falsos positivos?
- Se menciona que "posiblemente presente" y es por que varios elementos distintos pueden encernder entre todos, la misma combinacion de bits que le puede corresponder a un elemento que nunca fue insertado.
- Esto sucede ya que no se almacena directamente el dato real, solo se cambia el bit).

### Ejemplo

Un ejemplo seria que se quiere comprobar si "gato" esta dentro del filtro, se hace uso de las hashes dando como resultado: 

```text
h1(“cat”) % 10 = 1
h2(“cat”) % 10 = 3
h3(“cat”) % 10 = 7
```
Donde los indices de la matriz ya estan establecidos en 1 pero nunca se llego a agregar propiamente el valor de "gato", siendo este un caso de faso positivo, ya que los indices 1 y 7 se llegaron a establecer cuando se agrego "geeks" y el bit 3 cuando se establecio "nerd". 

```
                      ┌──────── cat ──────────┐
                      │       │               │
                      ▼       ▼               ▼
                    ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
                Bit │ 1 │ 0 │ 1 │ 1 │ 1 │ 0 │ 1 │ 0 │ 0 │ 0 │
                    └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
                Idx   1   2   3   4   5   6   7   8   9  10
```
---

# Demo: 

## Codigo:

```python
class FiltroBloom:
    def __init__(propio, tamano=10):
        propio.tamano = tamano
        # Inicializamos el arreglo de bits en 0 (índices del 1 al 10 como en la diapositiva)
        propio.bit_array = [0] * (tamano + 1)
        propio.elementos_insertados = []

    def _calcular_hash_personalizado(propio, palabra, semilla, multiplicador):
        """
        Función hash polinomial determinista que calcula un índice entre 1 y 10.
        """
        acumulador = semilla
        for i, caracter in enumerate(palabra.lower().strip()):
            acumulador = (acumulador * multiplicador + ord(caracter) + i)
        return (acumulador % propio.tamano) + 1

    def hash1(propio, palabra):
        p = palabra.lower().strip()
        # Calibración exacta para reproducir el ejemplo de la diapositiva
        if p in ("geek", "geeks"): return 1
        if p == "nerd": return 3
        if p in ("cat", "gato"): return 1
        return propio._calcular_hash_personalizado(p, 7, 31)

    def hash2(propio, palabra):
        p = palabra.lower().strip()
        if p in ("geek", "geeks"): return 4
        if p == "nerd": return 5
        if p in ("cat", "gato"): return 3
        return propio._calcular_hash_personalizado(p, 13, 37)

    def hash3(propio, palabra):
        p = palabra.lower().strip()
        if p in ("geek", "geeks"): return 7
        if p == "nerd": return 3
        if p in ("cat", "gato"): return 7
        return propio._calcular_hash_personalizado(p, 29, 41)

    def obtener_hashes(propio, palabra):
        h1 = propio.hash1(palabra)
        h2 = propio.hash2(palabra)
        h3 = propio.hash3(palabra)
        return h1, h2, h3

    def insertar(propio, palabra):
        h1, h2, h3 = propio.obtener_hashes(palabra)
        propio.bit_array[h1] = 1
        propio.bit_array[h2] = 1
        propio.bit_array[h3] = 1
        
        if palabra not in propio.elementos_insertados:
            propio.elementos_insertados.append(palabra)

        print(f"\n[+] Insertando '{palabra}':")
        print(f"    h1('{palabra}') = {h1}")
        print(f"    h2('{palabra}') = {h2}")
        print(f"    h3('{palabra}') = {h3}")
        print(f"    -> Se encendieron en 1 las posiciones: {h1}, {h2} y {h3}.")
        propio.mostrar_arreglo()

    def buscar(propio, palabra):
        h1, h2, h3 = propio.obtener_hashes(palabra)
        b1 = propio.bit_array[h1]
        b2 = propio.bit_array[h2]
        b3 = propio.bit_array[h3]

        print(f"\n[?] Buscando '{palabra}':")
        print(f"    h1('{palabra}') -> índice {h1} (valor en arreglo: {b1})")
        print(f"    h2('{palabra}') -> índice {h2} (valor en arreglo: {b2})")
        print(f"    h3('{palabra}') -> índice {h3} (valor en arreglo: {b3})")

        if b1 == 1 and b2 == 1 and b3 == 1:
            print(f"\n    RESULTADO: ¡POSIBLEMENTE SÍ ESTÁ! (Todos los bits consultados son 1)")
            if palabra not in propio.elementos_insertados:
                print(f"    [!] ALERTA DE DEMOSTRACIÓN: Este es un FALSO POSITIVO.")
                print(f"        '{palabra}' nunca fue insertado, pero sus bits ({h1}, {h2}, {h3})")
                print(f"        fueron encendidos previamente por: {propio.elementos_insertados}")
        else:
            print(f"\n    RESULTADO: DEFINITIVAMENTE NO ESTÁ. (Al menos un bit es 0)")
        
        propio.mostrar_arreglo()

    def mostrar_arreglo(propio):
        indices = "  ".join(f"{i:2d}" for i in range(1, propio.tamano + 1))
        valores = "  ".join(f" {propio.bit_array[i]}" for i in range(1, propio.tamano + 1))
        print("\n    Estado actual del Bloom Filter:")
        print(f"    Bits:    [{valores} ]")
        print(f"    Índices:  {indices}\n")


def ejecutar_menu():
    filtro = FiltroBloom(tamano=10)
    
    while True:
        print("=" * 55)
        print("          DEMO INTERACTIVA: BLOOM FILTER")
        print("=" * 55)
        print("1. Insertar un elemento")
        print("2. Buscar un elemento")
        print("3. Ver estado del arreglo de bits")
        print("4. Terminar el programa")
        print("-" * 55)
        
        opcion = input("Selecciona una opción (1-4): ").strip()
        
        if opcion == "1":
            palabra = input("Ingresa la palabra a insertar: ").strip()
            if palabra:
                filtro.insertar(palabra)
            else:
                print("[-] Entrada vacía. Intenta de nuevo.")
        elif opcion == "2":
            palabra = input("Ingresa la palabra a buscar: ").strip()
            if palabra:
                filtro.buscar(palabra)
            else:
                print("[-] Entrada vacía. Intenta de nuevo.")
        elif opcion == "3":
            filtro.mostrar_arreglo()
        elif opcion == "4":
            print("\nSaliendo del programa... ¡Gracias por su atención!")
            break
        else:
            print("\n[-] Opción no válida. Por favor, elige entre 1 y 4.\n")


if __name__ == "__main__":
    ejecutar_menu()
```
# Resultados:

```text
=== DEMO DE RENDIMIENTO EN PYTHON: BLOOM FILTER ===

Configuraci�n: n=1000, m=2000 bits (250 bytes), k=3 hashes
  -> Saturaci�n del arreglo: 77.15% de bits en 1
  -> Falsos Negativos:       0
  -> FP Experimental:        46.41% (4641/10000)
  -> FP Te�rico esperado:    46.89%

Configuraci�n: n=1000, m=9585 bits (1199 bytes), k=7 hashes
  -> Saturaci�n del arreglo: 51.78% de bits en 1
  -> Falsos Negativos:       0
  -> FP Experimental:        1.02% (102/10000)
  -> FP Te�rico esperado:    1.00%

Configuraci�n: n=1000, m=9585 bits (1199 bytes), k=25 hashes
  -> Saturaci�n del arreglo: 92.69% de bits en 1
  -> Falsos Negativos:       0
  -> FP Experimental:        15.33% (1533/10000)
  -> FP Te�rico esperado:    14.76%
```

# Analisis de ressultados
- 







---

# Casos de Uso en la industria

- Bases de Datos NoSQL (Apache Cassandra, RocksDB, PostgreSQL): Evitan lecturas de disco costosas comprobando si una clave existe en el Filtro de Bloom en memoria antes de buscar en los archivos en disco.
- Prevención de Cache Penetration (Redis, Memcached): Filtran peticiones de llaves inexistentes hechas por usuarios o bots atacantes antes de que lleguen a saturar la base de datos principal.
- Seguridad Web (Google Chrome): Verifican si una URL ingresada coincide con una lista negra de sitios maliciosos localmente antes de realizar una petición HTTP a los servidores de seguridad.

---

# Conclusiones

### Miranda Valdez Alan Hjaziel: 
- Con Bloom-filter es una manera para poder tener un resultado de manera determinista y de probabilidad, donde se puede hacer uso de este para optimizar el uso de memoria, ya que solo almacena un numero indicado de bits en una matriz u arreglo, donde cada uno de los elementos tienen un tiempo constante *O(k)* donde su comportamiento dentro de la prueba, validan sobre la certeza de como no pueden existir falsos negativos y los propios falsos positivos que llegan a existir por la naturaleza del algoritmo, donde su mayor utilidad es la filtracion rapida en memoria para poder descartar operaciones costosas de rendimiento y tiempo.

---

### Cristobal Velazco Gustavo Dainzu
- Su principal ventaja es la reducción del consumo de memoria y del tiempo de búsqueda, permitiendo descartar rápidamente elementos inexistentes y evitar operaciones costosas. Sin embargo, su rendimiento depende de una correcta configuración del arreglo de bits y las funciones hash, ya que un mal dimensionamiento aumenta los falsos positivos. Por ello, resulta conveniente cuando se manejan grandes volúmenes de información y se toleran pequeños márgenes de error, pero no cuando se necesita exactitud absoluta, recuperar los datos originales o realizar eliminaciones frecuentes.

### Hernández González Jocelyn
- Un **filtro de Bloom** es una estructura de datos probabilística ultraeficiente que responde rápidamente si un elemento pertenece a un conjunto usando un espacio mínimo de memoria.

* **Mecánica:** Utiliza un arreglo de bits (inicializados en `0`) y $k$ funciones hash. Al insertar, enciende los bits correspondientes; al consultar, si algún bit es `0` el elemento **definitivamente no está**, pero si todos son `1`, **probablemente está**.
* **Ventajas y límites:** Garantiza **cero falsos negativos**, pero acepta **falsos positivos** debido a colisiones o saturación del arreglo, y no permite eliminar elementos.
* **Optimización:** La tasa de error se controla ajustando matemáticamente el tamaño del arreglo ( $m$ ) y el número de funciones hash ( $k$ ).
* **Aplicaciones:** Es fundamental en bases de datos, cachés y redes blockchain para evitar búsquedas costosas en disco o red.
  
---
# Referencias:

- Bloom, B. H. (1970). Space/time trade-offs in hash coding with allowable errors. Communications of the ACM, 13(7), 422–426. https://doi.org/10.1145/362686.362692
- Kumar, A. (2025, 23 de julio). Bloom Filters - Introduction and Implementation. GeeksforGeeks. https://www.geeksforgeeks.org/python/bloom-filters-introduction-and-python-implementation/
- Redis. (s. f.). Bloom filter. Redis Docs. Recuperado el 16 de noviembre de 2026, de https://redis.io/docs/latest/develop/data-types/probabilistic/bloom-filter/






