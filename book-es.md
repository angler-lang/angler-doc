# Introducción

# Marco Teórico

# Marco Tecnológico

# Marco Metodológico

# Desarrollo

Investigación [verde]
Diseño [azul]
Implementación [rojo]

*****

**NOTA**: Se usarán algunos ejemplos de tipos básicos para hacer obvia la evolución.

Comenzamos con algunas ideas básicas de qué buscabamos en el diseño de este lenguaje:

- con una definción compacta
- explícito
- para principiantes
- con tipos dependientes

## Expresiones, tipos y sus construcciones

Ya que el lenguaje debe tener tipos dependientes, se decidió desde un principio que un tipo es sencillamente una expresión, no un elemento especial del lenguaje, hace que en cualquier parte donde se espera un tipo, debería poder escribirse cualquier expresión, mientras que el chequeo de tipos pase sin errores.

Al hacer estos dos elementos uno solo, se definieron las diferentes construcciones posibles de expresiones.

- Aplicación de funciones
- Parentización
- Lambda funciones
- Pattern matching
- Alcance de declaraciones para una expresión (`where`, `let-in`)
- Cuantificadores universales y existenciales de tipos

## Tipos y funciones, ¿cerrados o abiertos?

Nos preguntamos porqué se ha hecho una diferencia tan clara en otros lenguajes de programación funcionales entre lo que es una función, un tipo y un constructor de tipo, así que comenzamos con la idea de que todos éstos fueran declarados de la misma manera.

```haskell
Bool : Type
True : Bool
False : Bool

not : Bool -> Bool
not True = False
not False = True
```

Ésta alternativa hacía que la única diferencia entre una función y un tipo es que no tenga declaraciones asociadas, pero el siguiente ejemplo nos dejó preguntándonos qué debería suceder:

```haskell
Bool : Type
True : Bool
False : Bool

not : Bool -> Bool
not True = False
not False = True

True = False
```

¿Ahora `True` deja de ser algo sobre lo que se puede hacer pattern matching, como en la función `not`?; como podemos ver, da paso a que un programa deje de funcionar con tan solo una línea sospechosa.

Otro ejemplo que da paso a problemas sería agregar un constructor a un tipo que no se debe:

```haskell
Bool : Type
True : Bool
False : Bool

not : Bool -> Bool
not True = False
not False = True

IDK : Bool

not IDK     -- ?
```

Aquí agregamos un nuevo constructor a el tipo `Bool` y aunque tenga sentido usarlo en la función `not`, pues espera algo del tipo `Bool`, el programa va a dar un error al correr, no al ser analizado por errores estáticos.

Así que se nos ocurrieron los tipos cerrados, éstos son tipos cuyos constructores son listados inmediatamente en un alcance nuevo, y no se le puede agregar más constructores.

```haskell
Bool : Type where
    True : Bool
    False : Bool

not : Bool -> Bool
not True = False
not False = True
```

Y ahora ninguno de las confusiones anteriores suedería, al declarar el `IDK` no habría problema, pero al llamar `not IDK`, el intepretador reportaría un error estático.

Luego al ver `where` puede ser utilizado al final de cualquier expresión, por lo que no era una palabra que pudieramos utilizar para introducir los constructores, así que optamos por la palabra `with`. También el cambio muestra la diferencia semántica entre `where` y `with`, con el `where` se introduce un alcance sólo disponible en la expresión de éste, mientras que el `with` introduce elementos en el alcance exterior a éste.

```haskell
Bool : t where t = Type
    with
        True : Bool
        False : Bool
```

Pero parecía interensante la idea de tipos que sus constructores pudieran ser agregados mientras se fueran necesitando, viene en mente un ejemplo de monedas del mundo con tan solo algunas monedas _base_, para que el usuario agregue las de interés.

```haskell
Currency : Type
USD : Nat -> Currency
EUR : Nat -> Currency

toUSD : Currency -> Currency
toUSD (USD n) = USD n
toUSD (EUR n) = USD (n * 1.1)

-- ...

XBT : Nat -> Currency   -- bitcoin
toUSD (XBT n) = USD (n * 400)
```

Así que si tenemos tipos abiertos y cerrados, tendremos también funciones abiertas y cerradas; ya que estamos buscando enlazar lo más posible estos dos conceptos:

```haskell
not : Bool -> Bool where
    not True = False
    not False = True
```

Pero los tipos abiertos seguían sufriendo de los problemas presentados al principio, habría que hacer una diferencia más explícita entre tipos y funciones.

Los tipos, abiertos y cerrados, deben indicar explícitamente cuáles son sus constructores de tipos para evitar el problema antes mencionado, de forma que llegamos a la siguiente sintaxis:

```haskell
closed Bool : Type with
    True : Bool
    False : Bool

not : Bool -> Bool
not True = False
not False = True

closed Nat : Type with
    Z : Nat             -- zero
    S : Nat -> Nat      -- succ

open Currency : Type with
    USD : Nat -> Currency
    EUR : Nat -> Currency

toUSD : Currency -> Currency
toUSD (USD n) = USD n
toUSD (EUR n) = USD (n * 1.1)

-- ...

reopen Currency with
    XBT : Nat -> Currency

toUSD (XBT n) = USD (n * 400)
```

Además todo este análisis nos hizo darnos cuenta de que aunque los tipos, constructores de tipos y funciones compartan muchas cualidades, no son lo mismo, se diferencian semánticamente y ésto debería mostrarse en su definición también.

Haciendo finalmente claras las diferencias, dejamos de un lado la idea de funciones cerradas; aunque éstas pueden ser teóricamente interesantes, no conseguimos gran utilidad práctica, ya que de querer _cerrar_ una función abierta, simplemente se deben escribir todos los casos posibles de sus argumentos.

```haskell
isUSD : Currency -> Bool
isUSD (USD _) = True
isUSD _ = False         -- closes the function

-- ...

isUSD (XBT _) = True    -- will never run
```

## Identificadores, ¿mayúsculas o minúsculas?

En un principio se consideró usar la convención de Haskell, que los constructores de tipos y tipos se escribieran con la primera letra mayúscula, y que los valores con la primera letra minúscula; de esta manera se podía distinguir sintácticamente entre ellos.

Pero esta idea tenía algunos problemas, por ejemplo, los identificadores que comenzaran con símbolos y no letras, ¿a qué clase pertenecerían?, además debemos recordar que con tipos dependientes, los tipos son expresiones como cualquier otra, por lo que una distinción sintáctica entre el uso de tipos y valores no era conveniente.

## Cuantificadores: parámetros y argumentos implícitos

Como se busca un lenguaje explícito, todo identificador que se utilice en una expresión, debe estar previamente definido con un tipo acompañante, tomando inspiración de otros lenguajes con tipos dependientes llegamos a la idea de tener dos secciones de la firma de una función, la parte _implícita_, que es el cuantificador universal; y la parte _explícita_:

```haskell
compose : { (a:Type) -> (b:Type) -> (c:Type) } -> (b -> c) -> (a -> b) -> a -> c
```

Donde se escribe la parte implícita entre llaves (`{` y `}`) como un tipo más, y luego se escribe la parte explícita, donde todos los nombres introducidos en la parte implícita están disponibles. Esto nos dejaba con dudas acerca de cómo se podían escribir con una estructura similar los tipos existenciales.

Los argumentos implícitos se pasan a la función al comenzar su nombre con un caracter _piso_ (`_`):

```haskell
compose _Bool : { (b : Type) -> (c : Type) } -> (b -> c) -> (Bool -> b) -> Bool -> c
```

Con esta sintaxis los parámetros implícitos son __posicionales__, es decir, se pasan los argumentos con un orden específico. Esta manera de pasar argumentos implícitos nos da una restricción en los identificadores que se puedene escribir en el lenguaje, ya que ninguno podría comenzar con piso para poder diferenciarlos de argumentos implícitos.

Pero hay un mayor problema con la propuesta anterior, recordemos que en la teoría de tipos dependientes tenemos cuantificadores universales y existenciales; en un cuantificador universal no importa el orden de las variables declaradas, se puede especificar un valor para cualquera de las variables disponibles, es decir, los parámetros implícitos deberían ser todos accesibles para el pasaje de argumentos implícitos. Así llegamos a otra propuesta que separa completamente la sintaxis de firmas y de cuantificadores:

```haskell
compose : { a:Type, b:Type, c:Type } (b -> c) -> (a -> b) -> a -> c
```

Con esta nueva sintaxis podemos pasar argumentos implícitos en el orden que sea necesario:

```haskell
compose { b = Nat, c = Bool } : { a:Type } (Nat -> Bool) -> (a -> Nat) -> a -> Bool
```

Con esta nueva los parámetros implícitos son __nombrados__, es decir, se pasan los argumentos en el orden que nos convenga, sin importar su posición, y los parámetros explícitos se mantienen posicionales.

Finalemente decidimos hacer un último cambio a esta sintaxis para hacer obvio que se trata de un cuantificador:

```haskell
compose : forall a:Type, b:Type, c:Type . (b -> c) -> (a -> b) -> a -> c
```

Ahora la definición se asemeja mucho más a un cuantificador, haciendo más explícito su uso, y además podemos acceder a todas las variables declaradas y especificarlas.

Se consideró usar la palabra `product` en vez de `forall`, pero se decidió ir por una referencia directa a lógica simbólica, aunque estos se consideren los tipos producto. De igual forma, se definen los tipos existenciales con la palabra `exists`, aunque se consideró la palabra `sum`:

```haskell
vfilter : forall t:Type, n:Nat . (t -> Bool) -> Vect n t -> exists m:Nat . Vect m t
```

Logrando una estructura parecida entre los cuantificadores universales y existenciales. El cuantificador existencial __no__ puede recibir argumentos implícitos. Esta sintaxis es _azúcar sintáctica_ para los cuantificadores existenciales, ya que estos están definidos en el lenguaje.

## El _cuantificador_ de selección

Luego nos dimos cuenta de que podemos querer introducir nombres para parámetros explícitos, es decir, lo que normalmente haríamos con un cuantificador universal, pero en vez de ser _para todo_ que sea para el valor dado. Esta idea se maneja en los lenguajes estudiados colocando una asociación de nombre directamente:

```haskell
id' : (t : Type) -> t -> t
id' _ x = x
```

Se entiende, pero nos pareció poco intuitivo, así que preferimos agregarle la palabra `select` para hacer obvio que se acerca al funcionamiento de los cuantificadores.

```haskell
id' : (select t : Type) -> t -> t
id' _ x = x
```

Además se lee como _«selecciona un **Type**, que llamaremos **t**»_.

## Identificadores: evitando confusiones

Desde un principio se aceptó la idea de poder usar símbolos en los identificadores como `+`, `*` y demás, pero para facilitar el uso del lenguaje, se decidió que tendríamos dos categorías de caracteres, una para símbolos y otra para letras. De manera que ciertos caracteres, a pesar de estar juntos, se tomarían como identificadores separados:

```haskell
var         -- `var`
v0          -- `v0`
0v0         -- `0` `v0`
++          -- `++`
var++v0     -- `var` `++` `v0`
```

## No me importa, te lo digo en inglés, I _don't care_

En una función puede haber casos en que un argumento no es importante para su valor resultante, a estos argumentos se les puede asignar un nombre que no usaremos, o indicar explícitamente que no es importante.

```haskell
const : forall t:Type, v:Type . t -> v -> t
const x ? = x
```

Donde el caracter `?` indica que no importa ese valor, luego se decidió cambiar este caracter a `_`, ya que el signo de interrogación puede transmitir la idea de que __no sabemos__ qué será el argumento, cuando lo que se quiere transmitir es que __no importa__. Se eligió `_` porque es el caracter típicamente usado por otros lenguajes para lo miso, y visualmente, al llamar tan poco la atención transmite su poca importancia.

```haskell
const : forall t:Type, v:Type . t -> v -> t
const x _ = x
```

## ¿Identificador o caracter?

Los caracteres literales suelen escribirse usando comillas simples (`'`), así que se decidió que ningún identificador puede empezar con una comilla simple, ya que se estaría esperando un caracter literal. Comilla simple es el único _símbolo_ que está en ambas categorías de identificadores, la de símbolos y la de letras.

```haskell
'a'     -- caracter
a''     -- identificador
+'      -- identificador
```

## Operadores ahuecados

Un lenguaje didáctico sin operadores puede hacer curva de aprendizaje muy _empinada_, por lo que la posibilidad de definir operadores es escencial. Se propuso usar _operadores mixfijos_, estos funcionan para definir las __partes__ de un operador y sus __huecos__, que es donde irían sus operandos; para indicar los huecos de un operador, usaremos el caracter piso (`_`).

```haskell
if_then_else_ : forall t:Type . Bool -> t -> t -> t
if True  then x else _ = x
if False then _ else y = y
```

Esto nos crea una restricción sobre los identificadores, estos no pueden contener dos `_` seguidos, ya que no sabríamos dónde termina un argumento y comienza el siguiente.

Un identificador con `_` puede usarse como operador, colocando los argumentos en los huecos, o como una función normal escribiendo el identificador con los `_` incluidos.

```haskell
if_then_else_ : forall t:Type . Bool -> t -> t -> t
if_then_else_ True  x _ = x
if_then_else_ False _ y = y
```

Nótese que se usa `_` tanto para _don't care_ como para los huecos de los identificadores, e.g. `if_then_else_ False _ y`.

## Huecos entre identificadores

Gracias a la nueva sintaxis para operadores, se decidió que un las partes distintas de un identificador podrían ser de distintas categorías, es decir, pudiendo mezclarlas sólo cuando hay un `_` de por medio.

```haskell
if_?_:_ : forall t:Type . Bool -> t -> t -> t
if True  ? x : _ = x
if False ? _ : y = y
```

## Definición de operadores

En un principio se consideró que si un identificador tiene `_` se consideraría automáticamente un operador y tendría una precedencia y asociatividad por defecto, pero decidimos abandonar esa idea, pues podría generar comportamientos inesperados por usuarios. Así que llegamos a una definición para estos, donde no importa si el identificador es de una función o no, o está definido o no. Lo que se expresa es una regla de _reordenamiento_ de identificadores.

```haskell
operator |_|                closed
operator if_then_else_      prefix  0
operator _==_               infixN  2
operator _+_                infixL  4
operator _*_                infixL  5
operator _^_                infixR  6
operator -_                 prefix  7
operator _!                 postfix 8

-- ...

if | a * b + c | == d then e ! else - e
-- if_then_else_ (_==_ (|_| (_+_ (_*_ a b) c)) d) (_! e) (-_ e)
```

## Comportamientos

# Resultados

## Diseño

## Implementación

# Aplicaciones

# Concluciones

## Resumen de qué se hizo

## Porqué el resultado es positivo

## Trabajo futuro

# Apéndice

## Clases

## Extensiones

### Comportamientos
#### Azúcar sintáctica de monads

### Lambdas

### Literales polimórficos -- [1,2,3], "string, text"

## Semántica operacional

## Gramática

## ...
