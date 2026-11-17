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

# Funcionamiento del filtro Bloom
- Un filtro Bloom vacio es una matriz que contiene una matriz de m cantidad de bits:
  <img width="472" height="125" alt="image" src="https://github.com/user-attachments/assets/9d767ed0-a091-4f10-a81f-f41b93bd5e3c" />
                    ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
                Bit │ 1 │ 0 │ 0 │ 1 │ 0 │ 0 │ 1 │ 0 │ 0 │ 0 │
                    └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
                Idx   1   2   3   4   5   6   7   8   9  10

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

<img width="433" height="191" alt="image" src="https://github.com/user-attachments/assets/bd92271a-ad92-4af5-b538-6079acf8d400" />

---

# Demo: 

## Codigo:

```python
import hashlib
import math

class BloomFilter:
    def __init__(self, m: int, k: int):
        self.m = m  # Tamaño total en bits
        self.k = k  # Número de funciones hash[cite: 1]
        # Arreglo empaquetado de bytes (1 bit real por posición, 8 bits por byte)
        self.bit_array = bytearray((m + 7) // 8)

    def _set_bit(self, index: int) -> None:
        self.bit_array[index // 8] |= (1 << (index % 8))

    def _get_bit(self, index: int) -> bool:
        return (self.bit_array[index // 8] & (1 << (index % 8))) != 0

    def _get_indices(self, item: str):
        # Técnica Kirsch-Mitzenmacher (Double Hashing) usando SHA-256 dividido en dos enteros de 64 bits
        digest = hashlib.sha256(item.encode('utf-8')).digest()
        h1 = int.from_bytes(digest[:8], 'big')
        h2 = int.from_bytes(digest[8:16], 'big')
        
        for i in range(self.k):
            yield (h1 + i * h2) % self.m

    def insert(self, item: str) -> None:
        for index in self._get_indices(item):
            self._set_bit(index)

    def contains(self, item: str) -> bool:
        return all(self._get_bit(index) for index in self._get_indices(item))

    def bit_saturation(self) -> float:
        set_bits = sum(byte.bit_count() for byte in self.bit_array)
        return set_bits / self.m


def ejecutar_experimento(n: int, m: int, k: int, num_pruebas: int = 10000) -> None:
    bf = BloomFilter(m=m, k=k)

    # 1. Fase de inserción (elementos conocidos)
    insertados = [f"user_id_{i}" for i in range(n)]
    for item in insertados:
        bf.insert(item)

    # 2. Verificación de falsos negativos (siempre debe ser 0)[cite: 1]
    falsos_negativos = sum(1 for item in insertados if not bf.contains(item))

    # 3. Verificación de falsos positivos (elementos nunca insertados)[cite: 1]
    no_insertados = [f"guest_id_{i}" for i in range(num_pruebas)]
    falsos_positivos = sum(1 for item in no_insertados if bf.contains(item))

    # Métricas experimentales vs teóricas
    tasa_exp = (falsos_positivos / num_pruebas) * 100
    tasa_teo = ((1 - math.exp(-k * n / m)) ** k) * 100
    saturacion = bf.bit_saturation() * 100

    print(f"Configuración: n={n}, m={m} bits ({len(bf.bit_array)} bytes), k={k} hashes")
    print(f"  -> Saturación del arreglo: {saturacion:.2f}% de bits en 1")
    print(f"  -> Falsos Negativos:       {falsos_negativos}")
    print(f"  -> FP Experimental:        {tasa_exp:.2f}% ({falsos_positivos}/{num_pruebas})")
    print(f"  -> FP Teórico esperado:    {tasa_teo:.2f}%\n")


if __name__ == "__main__":
    print("=== DEMO DE RENDIMIENTO EN PYTHON: BLOOM FILTER ===\n")

    # Escenario A: Arreglo pequeño (subdimensionado)[cite: 1]
    ejecutar_experimento(n=1000, m=2000, k=3)

    # Escenario B: Dimensionamiento óptimo para p = 1% (0.01)[cite: 1]
    ejecutar_experimento(n=1000, m=9585, k=7)

    # Escenario C: Exceso de funciones hash (k demasiado alto)[cite: 1]
    ejecutar_experimento(n=1000, m=9585, k=25)

```
# Resultados:
<img width="590" height="375" alt="image" src="https://github.com/user-attachments/assets/c59904b5-9d20-4235-b167-5392a6abbd26" />

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


### 





---

