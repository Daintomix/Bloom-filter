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

Se quiere introducir la palabra "sol" donde se hace uso de las funciones hash con un array de longitud 10.
Primero se debe calcular los hashes:

```text
h1(“sol”) % 10 = 3
h2(“sol”) % 10 = 5
h3(“sol”) % 10 = 9
```

Ahora con estas salidas los bits entran en los indices 3,5,9 dentro de la matriz y se cambian a 

```
                                          sol
                                  │        │              │
                                  ▼        ▼              ▼
                    ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
                Bit │ 0 │ 0 │ 0 │ 1 │ 0 │ 1 │ 0 │ 0 │ 0 │ 1 │
                    └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
                Idx   1   2   3   4   5   6   7   8   9   10
```
            
Ahora de nuevo se queire introducir un dato, en este caso "ave", donde igualmente se calculara mediante hashes

```text
h1(“ave”) % 10 = 3
h2(“ave”) % 10 = 7
h3(“ave”) % 10 = 9
```

Donde ahora los bits a modificar seran 3, 7,9 dentro de la matriz
```
                                              ave
                                  │               │       │
                                  ▼               ▼       ▼
                    ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
                Bit │ 0 │ 0 │ 0 │ 1 │ 0 │ 1 │ 0 │ 1 │ 0 │ 1 │
                    └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
                Idx   1   2   3   4   5   6   7   8   9   10


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

Un ejemplo seria que se quiere comprobar si "rana" esta dentro del filtro, se hace uso de las hashes dando como resultado: 

```text
h1(“rana”) % 10 = 1
h2(“rana”) % 10 = 3
h3(“rana”) % 10 = 7
```
Donde los indices de la matriz ya estan establecidos en 1 pero nunca se llego a agregar propiamente el valor de "rana", siendo este un caso de faso positivo, ya que el indice 5 se llegaron a establecer cuando se agregó "sol" y los bits 7 y 9 cuando se establecio "ave". 

```
                                                rana
                                          │       │       │
                                          ▼       ▼       ▼
                    ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
                Bit │ 0 │ 0 │ 0 │ 1 │ 0 │ 1 │ 0 │ 1 │ 0 │ 1 │
                    └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
                Idx   1   2   3   4   5   6   7   8   9   10
```
---

# Demo: 

## Codigo:

```python
class FiltroBloom:
    def __init__(self, tamano=10):
        self.tamano = tamano
        self.bit_array = [0] * (tamano + 1)
        self.elementos_insertados = []

    def _obtener_hash(self, palabra, semilla):
        """
        Función hash determinista basada en la suma de los valores ASCII de las letras.
        Al ser matemática y predecible, permite demostrar colisiones exactas.
        """
        suma_ascii = sum(ord(c) for c in palabra.lower().strip())
        return ((suma_ascii * semilla) % self.tamano) + 1

    def obtener_hashes(self, palabra):
        p = palabra.lower().strip()
        h1 = self._obtener_hash(p, semilla=1)
        h2 = self._obtener_hash(p, semilla=2)
        h3 = self._obtener_hash(p, semilla=3)
        return h1, h2, h3

    def insertar(self, palabra):
        h1, h2, h3 = self.obtener_hashes(palabra)
        self.bit_array[h1] = 1
        self.bit_array[h2] = 1
        self.bit_array[h3] = 1
        
        if palabra not in self.elementos_insertados:
            self.elementos_insertados.append(palabra)

        print(f"\n[+] Insertando '{palabra}':")
        print(f"    h1('{palabra}') = {h1}")
        print(f"    h2('{palabra}') = {h2}")
        print(f"    h3('{palabra}') = {h3}")
        print(f"    -> Se encendieron en 1 las posiciones: {h1}, {h2} y {h3}.")
        self.mostrar_arreglo()

    def buscar(self, palabra):
        h1, h2, h3 = self.obtener_hashes(palabra)
        b1 = self.bit_array[h1]
        b2 = self.bit_array[h2]
        b3 = self.bit_array[h3]

        print(f"\n[?] Buscando '{palabra}':")
        print(f"    h1('{palabra}') -> índice {h1} (valor en arreglo: {b1})")
        print(f"    h2('{palabra}') -> índice {h2} (valor en arreglo: {b2})")
        print(f"    h3('{palabra}') -> índice {h3} (valor en arreglo: {b3})")

        if b1 == 1 and b2 == 1 and b3 == 1:
            if palabra not in self.elementos_insertados:
                print(f"\n    RESULTADO: ¡POSIBLEMENTE SÍ ESTÁ! (FALSO POSITIVO)")
                print(f"    [!] ALERTA: '{palabra}' NUNCA fue insertada.")
                print(f"        Sus bits ({h1}, {h2}, {h3}) fueron encendidos por: {self.elementos_insertados}")
            else:
                print(f"\n    RESULTADO: ¡POSIBLEMENTE SÍ ESTÁ! (Todos los bits son 1)")
        else:
            print(f"\n    RESULTADO: DEFINITIVAMENTE NO ESTÁ. (Al menos un bit es 0)")

    def mostrar_arreglo(self):
        indices = "  ".join(f"{i:2d}" for i in range(1, self.tamano + 1))
        valores = "  ".join(f" {self.bit_array[i]}" for i in range(1, self.tamano + 1))
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
                print("[-] Entrada vacía.")
        elif opcion == "2":
            palabra = input("Ingresa la palabra a buscar: ").strip()
            if palabra:
                filtro.buscar(palabra)
                filtro.mostrar_arreglo()
            else:
                print("[-] Entrada vacía.")
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

El experimento demuestra empíricamente cómo el dimensionamiento de la memoria ( $m$ ) y el número de funciones hash ( $k$ ) determinan la precisión y saturación de un filtro de Bloom al almacenar $n=1000$ elementos.   

- **Garantía absoluta contra falsos negativos:** En las tres pruebas, los falsos negativos se mantienen estrictamente en 0. Esto confirma que, sin importar cuán mal dimensionado esté el filtro, nunca omitirá un elemento que sí fue insertado.
- **Impacto de la falta de memoria (Escenario A):** Asignar solo 2 bits por elemento ( $m=2000$ ) provoca que el arreglo se llene rápidamente hasta un 77.15%. Al haber tan pocos ceros disponibles, casi la mitad de las consultas de elementos inexistentes colisionan por azar, disparando los falsos positivos al 46.41%.
- **El punto de equilibrio matemático (Escenario B):** Aumentar la memoria a ~9.6 bits por elemento ( $m=9585$ ) y usar el número óptimo de hashes ( $k=7$ ) mantiene la saturación del arreglo prácticamente en el 50% (51.78%). Esta distribución uniforme entre ceros y unos maximiza la entropía de la información y logra cumplir la meta de diseño: una tasa de error real de apenas 1.02%.
- **La trampa del exceso de hashes (Escenario C):** Manteniendo exactamente la misma memoria óptima (1199 bytes), elevar las funciones hash de $k=7$ a $k=25$ resulta contraproducente. Cada inserción enciende demasiados bits, disparando la saturación al 92.69%. Aunque se requieren 25 coincidencias simultáneas para dar un falso positivo, la altísima densidad de unos multiplica el error más de 15 veces respecto al escenario óptimo (subiendo a 15.33%).
- **Validación del modelo teórico:** En todos los casos, los resultados experimentales sobre 10,000 pruebas coinciden casi a la décima con las predicciones matemáticas teóricas (46.41% vs. 46.89%, 1.02% vs. 1.00% y 15.33% vs. 14.76%), validando la calidad de distribución de la técnica Double Hashing con SHA-256 utilizada en el código.  





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






