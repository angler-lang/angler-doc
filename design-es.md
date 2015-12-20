# Angler

Angler es un lenguaje de programación funcional con un sistema de tipos dependientes orientado a la enseñanza de las matemáticas discretas.

## Definición de sintaxis

- `<iden>`: para un espacio en donde se puede escribir un identificador.
- `<listid>`: para identificadores de listas de exportación / importación.
    + `<iden>`
    + `closed <iden> [ ( <iden> [ , <iden> ..] ) ]`
    + `open <iden> [ ( <iden> [ , <iden> ..] ) ]`
    + `operator <iden>`
- `<expr>`: para una expresión.
- `<type>`: para una expresión que debe ser de tipo `Type`.
- `<arg>`: para argumentos.
    + `_` (_don't care_)
    + `<iden>`
    + `<expr>`

## Identificadores

Los identificadores en Angler tienen las siguientes reglas de construcción:

1. Los identificadores constan de una o más _partes_
2. Las _partes_ puden ser de las siguientes formas:
    1. Comienzan por una letra, seguida de cero o más letras, números o comillas simples. -- `alpha [alpha digit ']*`
    2. Comienzan por un símbolo, seguido de cero o más símbolo o comillas simples. -- `symbol [symbol ']*`

        _Símbolos_: ``- ! @ # $ % & * + / \ < = > ^ | ~ ? ` [ ] : ; .``

3. Entra partes debe haber un caracter _piso_ (`_`)
4. Puede haber un `_` al principio o final de un identificador
5. No debe haber dos o más `_` seguidos.

Ejemplos de identificadores válidos:

```haskell
_+_
length
if_?_:_
ab2_cd
```

Ejemplos de identificadores inválidos:

```haskell
_:2_
_>a_        -- identificador `_>` , e identificador `a_`
if__then_   -- dos `_` seguidos
```

### Identificadores reservados

Aquí una lista de identificadores que son reservados para el lenguaje y su porqué:

- `:`: Para la anotación de tipos
- `=`: Para definición de valores
- `\` y `->`: Para expresiones lambda
- `.`: Para separación de un cuantificador y su expresión
- `(` y `)`: Para uso típico de forzar asociación
- `{` y `}`: Para aplicación implícita
- `,`: Para listas de identificadores y listas de anotación de tipos
- `_`: Para denotar un valor que no es de interés (_don't care_)
- `export`: Para indicar la lista de exportaciones
- `import`: Para importaciones
- `as`: Para importaciones cualificadas
- `open`: Para declaración de tipos abiertos
- `reopen`: Para expansión de tipos abiertos
- `closed`: Para declaración de tipos cerrados
- `with`: Para indicar la lista de constructores de un tipo
- `where`: Para declaraciones en un alcance
- `let` y `in`: Para expresiones _let_
- `forall`: Para expresiones _forall_
- `exists`: Para expresiones _exists_
- `select`: Para expresiones _select_
- `behaviour`, etc.: Para comportamientos
- `operator`: Para definición de operadores
- `prefix`: Para definir un operador como asociativo prefijo (derecho)
- `postfix`: Para definir un operador como asociativo sufijo (izquierdo)
- `infixL`: Para definir un operador como asociativo izquierdo
- `infixR`: Para definir un operador como asociativo derecho
- `infixN`: Para definir un operador como no asociativo

## Elementos embebidos

Los siguientes identificadores están embebidos en el lenguaje:
- `Type`: El tipo universal, del que todos los demás tipos son _subtipos_
- `_->_`: Para la construcción de tipos de funciones, _de un tipo al otro tipo_.
    - El constructor de este tipo sería las lambda-funciones `\x -> e`.

## Módulo

Un módulo es el programa dentro de __un__ archivo de Angler. Puede incluir una sección de _exportación_, y también de _importación_, ambas opcionales.

### Exportación

Una exportación sirve para indicar qué partes de un módulo deben estar disponibles al importarlo en otro.

Si no se coloca la exportación, todo lo definido en el módulo estará disponible para importación.

```haskell
export ( [ <listid> [ , <listid> ..] ] )
```

Un ejemplo válido, donde se exporta el tipo cerrado `Nat` con sus constructores `Z` y `S`, la función `_+_`, el operador para ésta.

```haskell
export (closed Nat(Z, S), _+_, operator _+_)
```

Se puede exportar todos los constructores de los tipos, en vez de listando cada uno de estos, usando la sintaxis `(..)`.

### Importación

Una importación puede ser _cualificada_, es decir, agregando a sus identificadores un prefijo.

```haskell
import <listid>  [ as <listid> ] [ ( <listid> [ , <listid> ..] ) ]
```

Un ejemplo mostrando las cuatro formas de hacer una importación.

```haskell
import Nat                                        -- *
import Map   as Map                               -- Map.*
import Maybe        (closed Maybe(Just, Nothing)) -- Maybe, Just, Nothing
import List  as L   (lookup, foldr)               -- L.lookup, L.fodlr
```

## Comentarios

Los comentarios pueden comenzar en cualquier punto del programa.

```haskell
{-
    Function _+_
    Addition for Nat (Natural numbers)
-}
_+_ : Nat -> Nat -> Nat
Z + m = m                   -- base case of addition
S n + m = S (n + m)         -- recursive case
```

### Comentarios de una línea

Pueden comenzar en cualquier parte de una línea con `--` y abarcan el resto de ésta.

```haskell
-- esto es un comentario de una línea
Z + Z   -- esto es otro comentario
```

### Comentarios de varias líneas

Pueden comenzar en cualquier parte de una línea con `{-` y terminan al conseguir un `-}`; tienen anidación.

```haskell
{-
    Este comentario ocupa varias líneas
    {-
        Hay anidación
    -}
Z + Z
-}
```

## Funciones

Una función consta de dos partes, su _declaración_ y su(s) _definición(es)_; la definición es opcional.

Ejemplo, la función identidad:

```haskell
id : forall t:Type . t -> t         -- declaration
id x = x                            -- definition
```

### Declaración de funciones

Una declaración de función consta de un identificador para ésta y el tipo asociado a este identificador, separados por `:`.

```haskell
<iden> : <type>
```

En el ejemplo de la función identidad, la declaración es la línea `id : forall t:Type . t -> t`.

La función de composición puede tener la siguiente declaración:

```haskell
_._ : forall a:Type, b:Type, c:Type . (b -> c) -> (a -> b) -> (a -> c)
```

Nótese que el operador `->` es asociativo a la derecha, por lo que la siguiente declaración es equivalente:

```haskell
_._ : forall a:Type, b:Type, c:Type . (b -> c) -> (a -> b) -> a -> c
```

Esto declara el tipo que debe tener la función a ser definida.

### Definición de funciones

Una definición de función consta de su identificador, seguido por sus argumentos y la expresión que la _define_, separados por `=`.

```haskell
<iden> [ <arg> ..] = <expr>
```

En el ejemplo de la función identidad, la definición es la línea `id x = x`; donde se separa en la siguientes partes:

- __`id x`__` = x`: el identificador (`id`) y su único argumento `x`.
- `id x`__` = `__`x`: el `=` que separa la expresión que la define de sus argumentos.
- `id x = `__`x`__: la expresión que define a la función.

La función de composición puede tener la siguente definición:

```haskell
_._ f g x = f (g x)
```

Aquí tenemos el identificador `_._`, tres argumentos `f`, `g` y `x`, y la expresión `f (g x)`.

Si definimos un operador para `_._` (`operator _._ infixR 9`), podríamos definir la función composición de la siguiente manera:

```haskell
(f . g) x = f (g x)
```

#### Argumentos

Los argumentos pueden ser un identificador, una expresión de _pattern matching_, o un caracter _piso_ (`_`) para indicar que no importa su valor.

Al leer cada parte de un argumento, se busca en la tabla de símbolos del programa; si no se encuentra en ésta, se agrega como identificador para la expresión de definición; si sí se encuentra, se usa para pattern matching.

Ejemplo, la función para modificación de los elementos de una lista:

```haskell
map : forall a:Type, b:Type . (a -> b) -> List a -> List b
map _ Nil = Nil
map f (x :: xs) = f x :: map f xs
```

Un análisis por definición:
- __Primera definición `map _ Nil = ...`__
    + __Primer argumento `_`__: Se usa piso `_` para ignorar la función de modificación
    + __Segundo argumento `Nil`__: Se usa pattern matching para una lista de cero elementos
- __Segunda definición `map f (x :: xs) = ...`__
    + __Primer argumento `f`__: Se usa el identificador `f` para denotar la función de modificación
    + __Segundo argumento `x :: xs`__: Se usa pattern matching para una lista de uno o más elementos

### Lambda funciones

Son funciones que no tienen un identificador asociado a ellas.

```haskell
\ <arg> [ <arg> ..] -> <expr>
```

Algunos ejemplos:

```haskell
\ x -> x * 2            -- double
\ _ :: xs -> xs         -- tail
\ x _ -> x              -- const
```

### Aplicación de funciones

Para usar una función, se coloca su expresión seguida de su argumento, en caso de haber.

```haskell
<expr> <expr>
```

Por ejemplo, usando la función de composición:

```haskell
_._ even length
```

ésto deja un valor con el tipo `forall a:Type . List a -> Bool`, veamos porqué:

```haskell
even : Nat -> Bool
length : forall a:Type . List a -> Nat
_._ : forall a:Type, b:Type, c:Type . (b -> c) -> (a -> b) -> (a -> c)
```

al aplicar `_._ even` nos queda una función del tipo:

```haskell
_._ even : forall a:Type . (a -> Nat) -> (a -> Bool)
```

luego, al aplicar esa función a `length`, nos queda:

```haskell
_._ even length : forall a:Type . List a -> Bool
```

### Cuantificadores

Si queremos escribir una función para valores de un _estilo_, se usan cuantificadores.

#### Cuantificador _forall_

Para indicar tipos que cumplen con un _estilo_.

```haskell
forall <iden> : <type> [ , <iden> : <type> ..] . <expr>
```

Esto introduce los identificadores mencionados y con el tipo indicado.

Por ejemplo, la función identidad funciona para cualquier tipo:

```haskell
id : forall t:Type . t -> t
id x = x
```

Otro ejemplo es la función `vmap`, que funciona para vectores de cualquier tipo y tamaño:

```haskell
vmap : forall a:Type, b:Type, n:Nat . (a -> b) -> Vect n a -> Vect n b
vmap _ VNil = VNil
vmap f (x <::> xs) = f x <::> vmap f xs
```

##### Aplicación de parámetros implícitos

Este cuantificador es el único que puede recibir una aplicación implícita, ya que sirve _para todos_ los valores:

```haskell
<expr> { <iden> = <expr> }
```

Un ejemplo sería utilizarlo para reducir el dominio de una función:

```haskell
map {b = Bool} : forall a:Type . (a -> Bool) -> List a -> List Bool
```

#### Cuantificador _exists_

Para indicar que existe un valor de un tipo, aunque no sepamos cuál será aún.

```haskell
exists <iden> : <type> ; <expr>
```

Esto introduce el identificador mencionado y con el tipo indicado, pero retorna una _tupla_ con el valor calculado y el valor encontrado para el existencial.

Un ejemplo es la función para filtrar elementos de un vector:

```haskell
vfilter : forall a:Type, n:Nat . (a -> Bool) -> Vect n a -> exists m:Nat ; Vect m a
vfilter _ VNil = VNil
vfilter g (x <::> xs) = (if g x then _<::>_ x else id) (vfilter g xs)
```

El tipo existencial es:

```haskell
closed Exists : (select a:Type) -> (a -> Type) -> Type with
    <*_;_*> : forall a:Type, P:a -> Type . (select x:a) -> P x -> Exists a P
```

#### Cuantificador _select_

Este _cuantificador_ sirve para referirse al valor del tipo indicado más adelante.

```haskell
select <iden> : <type>
```

Un ejemplo es la función identidad con anotación de tipo que llamaremos `the`:

```haskell
the : (select t:Type) -> t -> t
the _ x = x
```

Esta función recibe el tipo que espera explícitamente, a diferencia de `id`, que lo recibe implícitamente.

Otro ejemplo es la conversión de lista a vector:

```haskell
ListToVect : forall t:Type . (select xs : List t) -> Vect (length xs) t
ListToVect Nil = VNil
ListToVect (x :: xs) = x <::> ListToVect xs
```

## Operadores

Los operadores sirven para usar funciones de manera natural; se definen con una asociatividad y precedencia, el identificador indica dónde espera los argumentos usando el caracter piso (`_`).

```haskell
operator <iden> <fixity> <prec>
```

donde `<fixity>` es una de las siguientes asociatividades: `postfix`, `prefix`, `infixL`, `infixR`, `infixN`; y `<prec>` es el número natural para indicar su precedencia.

Algunos ejemplos:

```haskell
operator _!            postfix 20
operator -_            prefix  19
operator _^_           infixR  17
operator _+_           infixL  15
operator _-_           infixL  15
operator _==_          infixN  10
operator _/=_          infixN  10
operator if_then_else_ prefix  5
```

Esto hará que expresiones que cumplan esas reglas, se _transformen_ a aplicaciones de funciones.

Por ejemplo:

```haskell
if a == b then - x else y       -- if_then_else_ (_==_ a b) (-_ x) y
- x! + y ^ 2 == z               -- _==_ (_+_ (-_ (_! x)) (_^_ y 2)) z
```

Sólo se buscará estos _patrones_ al definir un operador explícitamente, es decir, los identificadores con `_` en ellos no tienen una asociatividad y precedencia por defecto.

También se puede usar en la definición de funciones:

```haskell
if_then_else_ : forall t:Type . Bool -> t -> t -> t
if True  then x else _ = x      -- if_then_else_ True  x _ = x
if False then _ else y = y      -- if_then_else_ False _ y = y
```

## Tipos

En Anlger podemos declarar dos diferentes _tipos_ de tipos; los _cerrados_ y los _abiertos_.

### Tipos cerrados

Son tipos cuya definición completa es conocida, se usa la palabra reservada `closed` para su definción. Se define con un identificador, un tipo, y sus posibles constructores.

> El tipo `_->_` es un tipo cerrado embebido en el lenguaje.

```haskell
closed <iden> : <type> with
    <iden> : <type>
    [ <iden> : <type> ..]
```

Por ejemplo, el conjunto de los _booleanos_ es un tipo cerrado:

```haskell
closed Bool : Type with
    True : Bool
    False : Bool
```

También los números naturales son representables con un tipo cerrado:

```haskell
closed Nat : Type with
    Z : Nat         -- zero
    S : Nat -> Nat  -- succesor
```

Un ejemplo un poco más complejo es el de listas y de vectores:

```haskell
closed List : Type -> Type with
    Nil : forall t:Type . List t
    _::_ : forall t:Type . t -> List t -> List t

closed Vect : Nat -> Type -> Type with
    VNil : forall t:Type . Vect Z t
    _<::>_ : forall t:Type, n:Nat . t -> Vect n t -> Vect (S n) t
```

### Tipos abiertos

Los tipos abiertos se comportan exactamente igual que los tipos cerrados, pero sirve para conjuntos cuya definición completa es desconocida (o infinita).

> El tipo `Type` es un tipo abierto embebido en el lenguaje. Cada tipo nuevo se considera una _reapertura_ de éste.

#### Declaración de tipos abiertos

Al declarar un tipo abierto, puede no conocerse ningún constructor de éste, por lo que la parte de constructores es opcional.

```haskell
open <iden> : <type> [ with
    <iden> : <type>
    [ <iden> : <type> ..]
    ]
```

Un ejemplo de declaración donde conocemos algunos constructores de antemano es el de monedas:

```haskell
open Currency : Type with
    USD : Nat -> Currency
    EUR : Nat -> Currency
```

Un ejemplo en el que no conocemos los constructores de antemano:

```haskell
open Command : Type
```

#### Reapertura de tipos abiertos

Después de haber introducido un tipo abierto, puede reabrirse éste para indicar más constructores.

```haskell
reopen <iden> with
    <iden> : <type>
    [ <iden> : <type> ..]    
```

Reapertura del ejemplo de monedas, actualizando el módulo:

```haskell
reopen Currency with
    XBT : Nat -> Currency       -- Bitcoin
```

Reapertura del otro ejemplo, para agregar los primeros comandos:

```haskell
reopen Command with
    Run : Nat -> Command
    Hit : Weapon -> Command
    Jump : Command
```

## Comportamientos

Los comportamientos sirven para obtener un estilo de sobrecarga de funciones, esto es, usar un mismo identificador para diferentes funciones (con tipos diferentes).

### Definición

Se definen con un identificador; un _argumento_ con su tipo; las funciones que define; y sus dependencias, en caso de existir.

```haskell
behaviour <iden> on <iden> : <type> defines
        <iden> : <type>
        [ <iden> : <type> ..]
    [ with
        <iden> [ <arg> ..] = <expr>
        [ <iden> [ <arg> ..] = <expr> ..]
    ]
```

Por ejemplo, el comportamiento de las cosas que son _igualables_:

```haskell
behaviour Eq on t : Type defines
        _==_ : t -> t -> Bool
        _/=_ : t -> t -> Bool
    with
        a /= b = not (a == b)
        a == b = not (a /= b)
```

### Instanciación

Para definir que un tipo cumple con un comportamiento.

```haskell
<iden> is <iden> with
    <iden> [ <arg> ..] = <expr>
    [ <iden> [ <arg> ..] = <expr> ..]
```

Se lee «_tipo_ es un _comportamiento_, donde ...».

Las instancias de `Eq` para `Bool` y `Nat` serían:

```haskell
Bool is Eq with
    True == True = True
    False == False = True
    _ == _ = False

Nat is Eq with
    Z == Z = True
    S n == S m = n == m
    _ == _ = False
```

### Requerimiento

Al escribir un tipo, se puede requerir que un valor no sólo sea de un tipo, sino que además cumpla con algún comportamiento.

```haskell
<type> is <iden>
```

Un ejemplo, la función que indica si un elemento se encuentra dentro de una lista:

```haskell
elem : forall t : Type is Eq . t -> List t -> Bool
elem _ Nil = False
elem y (x :: xs) = if y == x then True else elem y xs
```

También se puede requerir comportamientos para la definición de otros:

```haskell
behaviour Semigroup on t : Type defines
    _<+>_ : t -> t -> t

behaviour Monoid on t : Type is Semigroup defines
    neutral : t
```

### Alcances de comportamientos

Podríamos querer hacer varias instancias de un comportamiento para el mismo tipo, podemos llevarlo a cabo usando alcances de comportamientos. Hay alcances _abiertos_ y _cerrados_.

#### Definción de alcances de comportamientos abiertos

Simplemente se le asigna un nombre.

```haskell
scope <iden>
```

Si se quiere que una instancia forme parte de un alcance abierto específico, se indica usando la palabra reservada `at`:

```haskell
<iden> is <iden> at <iden> with
    <iden> [ <arg> ..] = <expr>
    [ <iden> [ <arg> ..] = <expr> ..]
```

Por ejemplo, haremos alcances para las distintas instancias de `Semigroup` que puede tener `Bool`.

```haskell
scope All
scope Any

Bool is Semigroup at All with
    _<+>_ = _&&_
Bool is Semigroup at Any with
    _<+>_ = _||_

Bool is Monoid at All with
    neutral = True
Bool is Monoid at Any with
    neutral = False

```

#### Definición de alcances de comportamientos cerrados

Se asigna un nombre al alcance y se definen todas sus instancias inmediatamente.

```haskell
scope <iden> with

    <iden> is <iden> with
        <iden> [ <arg> ..] = <expr>
        [ <iden> [ <arg> ..] = <expr> ..]

    [ <iden> is <iden> with
        <iden> [ <arg> ..] = <expr>
        [ <iden> [ <arg> ..] = <expr> ..]
    ..]
```

Por ejemplo, haremos alcances para las distintas instancias de `Semigroup` que puede tener `Bool`.

```haskell
scope All with
    Bool is Semigroup with
        _<+>_ = _&&_

    Bool is Monoid with
        neutral = True

scope Any with
    Bool is Semigroup with
        _<+>_ = _||_

    Bool is Monoid with
        neutral = False
```

#### Indicación de alcance de comportamientos

Para usar un alcance específico en una expresión, se debe indicar usando la palabra reservada `at`. Buscará todas las instacias en el alcance indicado, y si no las consiguió en el alcance por defecto.

```haskell
<expr> at <iden>
```

Un ejemplo:

```haskell
isNeutral : forall t : Type is Eq, Monoid . t -> Bool
isNeutral x = x == neutral

isNeutral True          -- error: no Monoid instance
isNeutral True at All   -- True
isNeutral True at Any   -- False
```

## Modo de pruebas

___TODO___

## Azúcar Sintáctica

Hay ciertas construcciones en el lenguaje a las que se les podría dar una sintaxis especial para facilitar su escritura, esto lo llamamos _azúcar sintáctica_.

### Listas

La versión _azucarada_ de listas sería `[0, 1, 2]`, para ser _desazucarado_ a `0 :: 1 :: 2 :: Nil`.

Aunque esto se puede lograr agregando algunos operadores y funciones, debemos considerar que `,` no es un símbolo utilizable para identificadores:

```haskell
operator [_  prefix  9
operator  _] postfix 10

[_ : forall t:Type . List t -> List t
[_ = id

_] : forall t:Type . t -> List t
xs ] = xs :: Nil

sugarTest : List Nat
sugarTest = [ 0 :: 1 :: 2 ]
```

### Cadenas de caracteres

La versión _azucarada_ de cadenas de caracteres (`String`) serían todos los caracteres seguidos entre dos comillas doblres (`"`).

```haskell
String : Type
String = List Char
```

Por ejemplo, `"Text\n"` es _desazucarado_ a `'T' :: 'e' :: 'x' :: 't' :: '\n' :: Nil`.

### Notación `do`

La notación `do` se usa para _Applicative_:

```haskell
do
    x <- action0
    action2 x
    z <- action3
    action4 z x
```

Se convierte en:

```haskell
action0 >>= \x -> action2 x >> action3 >>= \z -> action4 z x
```
