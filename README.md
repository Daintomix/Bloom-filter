# Bloom-filter
- Exposicion sobre EDAII sobre bloom filter por MVAH, HGJ y CVGD

# Tema 2: Bloom filter

# Objetivo: Mostrar como un arreglo de buts y varias funciones hash responden "¿esta?" con muy poco espacio, a cambio de aceptar falsos positivos
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

Se necesitan k funciones de hash para que se pueda calcular los hashes de una entrada. Donde cuando se añade un elemento al filtro se activan en los indices h1(x), h2(x) ... hk(x) donde en los indeces se calcula por la funcion hash.



## Ejemplo

Se quiere introducir la palabra "geeks" donde se hace uso de las funciones hash con un array de longitud 10.
Primero se debe calcular los hashes:

```text
h1(“geeks”) % 10 = 1
h2(“geeks”) % 10 = 4
h3(“geeks”) % 10 = 7
```

Ahora con estas salidas los bits entran en los indices 1, 4, 7 dentro de la matriz y se cambian a 1
<img width="460" height="172" alt="image" src="https://github.com/user-attachments/assets/e29de23a-b0bd-451e-80b6-8aa83ba692dc" />

Ahora de nuevo se queire introducir un dato, en este caso "nerd", donde igualmente se calculara mediante hashes

```text
h1(“nerd”) % 10 = 3
h2(“nerd”) % 10 = 5
h3(“nerd”) % 10 = 4
```

Donde ahora los bits a modificar seran 3, 5, 4 dentro de la matriz

<img width="432" height="171" alt="image" src="https://github.com/user-attachments/assets/8ced9b20-075c-442d-b028-f2a455354130" />

---

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
# Casos de Uso en la industria

- Bases de Datos NoSQL (Apache Cassandra, RocksDB, PostgreSQL): Evitan lecturas de disco costosas comprobando si una clave existe en el Filtro de Bloom en memoria antes de buscar en los archivos en disco.
- Prevención de Cache Penetration (Redis, Memcached): Filtran peticiones de llaves inexistentes hechas por usuarios o bots atacantes antes de que lleguen a saturar la base de datos principal.
- Seguridad Web (Google Chrome): Verifican si una URL ingresada coincide con una lista negra de sitios maliciosos localmente antes de realizar una petición HTTP a los servidores de seguridad.



---
### Medición de rendimiento ($n = 100,000,000$) y comparación de `threshold`

Para evaluar el impacto del parámetro `threshold` frente a la versión secuencial (`max_subarray_seq`), se midió el tiempo de ejecución con un arreglo de $100,000,000$ de enteros ($\approx 381\text{ MB}$) variando el umbral de corte desde valores muy pequeños hasta el tamaño total del arreglo:

| Algoritmo / Configuración | `threshold` | Tareas generadas (aprox.) | Tiempo (s) | Speedup ($T_{\text{seq}} / T_{\text{par}}$) |
|---|---|---|---|---|
| `max_subarray_seq` (Secuencial) | N/A | 1 | 0.126 s | 1.00x |
| `max_subarray_par` | 100 | ~2,097,151 | 1.840 s | 0.07x |
| `max_subarray_par` | 1,000 | ~262,143 | 0.295 s | 0.43x |
| `max_subarray_par` | 10,000 | ~32,767 | 0.112 s | 1.13x |
| `max_subarray_par` | 100,000 | ~2,047 | 0.093 s | 1.35x |
| `max_subarray_par` | 1,000,000 | ~255 | 0.095 s | 1.33x |
| `max_subarray_par` | 10,000,000 | ~15 | 0.104 s | 1.21x |
| `max_subarray_par` | 100,000,000 | 1 | 0.126 s | 1.00x |

*Nota: Con `threshold <= 1,000`, la sobrecarga de creación de tareas hace que la versión paralela sea más lenta que la secuencial (`speedup < 1.0`). El punto óptimo se alcanza entre `100,000` y `1,000,000` de elementos por tarea, logrando un speedup de ~`1.33x - 1.35x`.*

---

### Análisis Trabajo-Profundidad ($T_1$, $T_\infty$ y $\Pi$)

Considerando `threshold = 1` y que cada llamada a `combine` cuenta como $1$ paso (mientras que los casos base de $1$ elemento son inmediatos y cuestan $0$ pasos de combinación):

#### 1. Cálculo para un arreglo de $n = 8$ elementos
Al dividir recursivamente un arreglo de $8$ elementos a la mitad hasta llegar a hojas de tamaño $1$, se forma un árbol binario completo de altura $\log_2(8) = 3$:
* **Nivel 1 (inferior):** Se unen los $8$ elementos individuales en $4$ pares $\rightarrow 4$ operaciones `combine` (pueden ejecutarse en paralelo en $1$ paso de profundidad).
* **Nivel 2 (intermedio):** Se unen los $4$ pares en $2$ cuartetos $\rightarrow 2$ operaciones `combine` ($1$ paso de profundidad).
* **Nivel 3 (raíz):** Se unen las $2$ mitades de tamaño $4$ en el resultado final $\rightarrow 1$ operación `combine` ($1$ paso de profundidad).

Por lo tanto, para $n = 8$:
* **Trabajo ($T_1$):** Suma total de operaciones `combine` en todos los niveles:
  $$T_1(8) = 4 + 2 + 1 = 7\text{ pasos}$$
* **Profundidad ($T_\infty$):** Camino crítico o cantidad de niveles secuenciales de combinaciones:
  $$T_\infty(8) = \log_2(8) = 3\text{ pasos}$$
* **Paralelismo ($\Pi$):**
  $$\Pi(8) = \frac{T_1(8)}{T_\infty(8)} = \frac{7}{3} \approx 2.33$$

#### 2. Generalización para $n$ elementos (asumiendo $n = 2^k$)
* **Trabajo ($T_1$):** Un árbol binario con $n$ hojas tiene exactamente $n - 1$ nodos internos (cada nodo interno es un `combine`):
  $$T_1(n) = \sum_{i=0}^{\log_2(n)-1} 2^i = n - 1 = \Theta(n)$$
* **Profundidad ($T_\infty$):** En cada nivel del árbol todas las llamadas a `combine` son independientes entre sí, por lo que cada nivel aporta $1$ paso a la profundidad total:
  $$T_\infty(n) = \log_2(n) = \Theta(\log n)$$
* **Paralelismo ($\Pi$):**
  $$\Pi(n) = \frac{T_1(n)}{T_\infty(n)} = \frac{n - 1}{\log_2(n)} = \Theta\left(\frac{n}{\log n}\right)$$

| Caso | Trabajo ($T_1$) | Profundidad ($T_\infty$) | Paralelismo ($\Pi = T_1 / T_\infty$) |
|---|---|---|---|
| $n = 8$ (`threshold = 1`) | $7$ | $3$ | $7 / 3 \approx 2.33$ |
| General $n$ (`threshold = 1`) | $n - 1 \in \Theta(n)$ | $\log_2(n) \in \Theta(\log n)$ | $\frac{n - 1}{\log_2(n)} \in \Theta\left(\frac{n}{\log n}\right)$ |
| General $n$ (`threshold = k`) | $\Theta(n)$ | $\Theta\left(k + \log_2\left(\frac{n}{k}\right)\right)$ | $\Theta\left(\frac{n}{k + \log_2(n/k)}\right)$ |

---

### Preguntas de análisis

* **¿Qué cambia en la práctica cuando `threshold` es grande?**
  Cuando el `threshold` ($k$) crece, el árbol recursivo se trunca antes de llegar a elementos individuales, generando únicamente $\approx \frac{n}{k}$ hojas. Cada hoja procesa un bloque contiguo de hasta $k$ elementos usando `max_subarray_seq` en un bucle lineal sin crear más tareas. Esto reduce drásticamente el número de operaciones `combine` y de tareas creadas a $\frac{n}{k} - 1$, aprovecha la localidad espacial de la memoria caché (lectura secuencial) y aumenta la profundidad a $T_\infty = \Theta(k + \log_2(n/k))$. Si `threshold` es demasiado grande (cercano a $n$), se crean muy pocas tareas ($\frac{n}{k} < P$, donde $P$ es el número de hilos), dejando núcleos ociosos y perdiendo paralelismo hasta comportarse igual que la versión secuencial cuando $\text{threshold} \ge n$.

* **¿Por qué un `threshold` muy pequeño es más lento que la versión secuencial?**
  Porque el costo de gestión de tareas (*overhead* del *runtime* de OpenMP) supera por mucho al trabajo útil de cada tarea. Crear una tarea con `#pragma omp task` implica reservar memoria para su contexto (variables `firstprivate`), encolarla en el planificador de hilos y sincronizarla en el `taskwait`, lo cual consume miles de ciclos de CPU. En cambio, `combine` o unas pocas iteraciones de `max_subarray_seq` requieren apenas unas cuantas instrucciones aritméticas. Con `threshold = 1` o `100` en un arreglo de $100$ millones de elementos, se instancian millones de tareas diminutas y el procesador pasa más del $95\%$ del tiempo administrando colas de tareas en lugar de sumar elementos del arreglo.

* **¿Por qué el speedup no alcanza el número de hilos ($P$)?**
  1. **Cuello de botella de memoria (*Memory-bound*):** El algoritmo tiene una intensidad aritmética muy baja (apenas un par de sumas y comparaciones por cada `int` de 4 bytes leído desde la memoria RAM). Aunque varios núcleos trabajen simultáneamente, todos compiten por el mismo bus de memoria principal, saturando el ancho de banda de la RAM mucho antes de saturar la capacidad de cómputo de los núcleos.
  2. **Sobrecarga de paralelización y sincronización:** A pesar de usar un `threshold` adecuado, iniciar la región paralela, planificar las tareas entre los hilos y esperar en los `#pragma omp taskwait` introduce latencia adicional que no existe en `max_subarray_seq`.
  3. **Ley de Amdahl en la reducción del árbol:** En los niveles superiores del árbol recursivo (cerca de la raíz) hay menos subproblemas que hilos disponibles (por ejemplo, en el último `combine` solo trabaja $1$ hilo mientras los demás esperan), reduciendo la eficiencia global.

