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

