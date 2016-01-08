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
- `behaviour`, `on`, `defines`, `is`: Para comportamientos
- `operator`: Para definición de operadores
- `prefix`: Para definir un operador como asociativo prefijo (derecho)
- `postfix`: Para definir un operador como asociativo sufijo (izquierdo)
- `infixL`: Para definir un operador como asociativo izquierdo
- `infixR`: Para definir un operador como asociativo derecho
- `infixN`: Para definir un operador como no asociativo

## Elementos embebidos

Los siguientes elementos están embebidos en el lenguaje:

- `Type`: El tipo universal, del que todos los demás tipos son _subtipos_, `Type` es _subtipo_ de sí mismo.
- `_->_`: Para la construcción de tipos de funciones, _de un tipo al otro tipo_.
    - El constructor de este tipo sería las lambda-funciones `\x -> e`.
- `Exists`: El tipo existencial, sirve para construir un valor que depende de otro valor de un tipo dado.
- `<*_;_*>`: El constructor de tipo existencial, devuelve el valor y la construcción lograda con éste.

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

Pueden comenzar en cualquier parte de una línea con `{-` y terminan al conseguir un `-}`; los comentarios de varias líneas tienen anidación.

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

Dado a que el operador `->` es asociativo a la derecha, la siguiente declaración es equivalente:

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

Al leer cada parte de un argumento, se busca en la tabla de símbolos del programa; si no se encuentra en ésta, se agrega como identificador para la expresión de definición; si se encuentra, se usa para pattern matching.

Ejemplo, la función para modificación de los elementos de una lista:

```haskell
map : forall a:Type, b:Type . (a -> b) -> List a -> List b
map _ Nil = Nil
map f (x :: xs) = f x :: map f xs
```

Se hará un análisis por cada definición:
- __Primera definición `map _ Nil = ...`__
    + __Primer argumento `_`__: Se usa piso `_` para ignorar la función de modificación.
    + __Segundo argumento `Nil`__: Se usa pattern matching para reconocer una lista de cero elementos.
- __Segunda definición `map f (x :: xs) = ...`__
    + __Primer argumento `f`__: Se usa el identificador `f` para denotar la función de modificación.
    + __Segundo argumento `x :: xs`__: Se usa pattern matching para reconocer una lista de uno o más elementos, introduciendo los identificadores `x` y `xs` como _cabeza_ y _cola_ de la lista, respectivamente.

### Lambda funciones

Son funciones que no tienen un identificador asociado a ellas.

```haskell
\ <arg> [ <arg> ..] -> <expr>
```

Algunos ejemplos:

```haskell
\ x -> x * 2            -- double
\ (_ :: xs) -> xs       -- tail
\ x _ -> x              -- const
```

### Aplicación de funciones

Para usar una función, se coloca su expresión seguida de su argumento. La aplicación de funciones tiene asociatividad izquierda, es decir, en caso de haber varias aplicaciones seguidas, se aplican los dos más izquierdos y luego se aplica eso al siguiente valor a su derecha, y así sucesivamente.

```haskell
<expr> <expr>
```
Un ejemplo sencillo es la aplicación de negación booleana:

```haskell
not True : Bool     -- False
```

Un ejemplo más complicado con tres aplicaciones, usando la función de composición (`_._`), la función que indica si un número es par (`even`) y la función que indica la cantidad de elementos en una lista (`length`), obtenemos una función que calcula si la cantidad de elementos en una lista es par:

```haskell
_._ even length : forall a:Type . List a -> Bool
```

veamos porqué tiene ese tipo, tenemos que los identificadores utilizados tienen los tipos:

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

Para asociar identificadores a valores de un tipo a ser usados posteriormente en la firma.

#### Cuantificador _forall_

Para asociar identificadores a un valor arbitrario de un tipo dado, como generalización.

```haskell
forall <iden> : <type> [ , <iden> : <type> ..] . <expr>
```

Ésto introduce los identificadores mencionados y con el tipo indicado.

Por ejemplo, la función identidad funciona para cualquier tipo:

```haskell
id : forall t:Type . t -> t
id x = x
```

Otro ejemplo es la función `vmap`, que funciona para vectores, un vector es una lista a la que se le específica no sólo el tipo a contener, sino la cantidad de elementos dentro de ésta:

```haskell
vmap : forall a:Type, b:Type, n:Nat . (a -> b) -> Vect n a -> Vect n b
vmap _ VNil = VNil
vmap f (x <::> xs) = f x <::> vmap f xs
```

##### Parámetros implícitos

Son los identificadores asociados en un _forall_ (e.g. la `t` en `id : forall t:Type . t -> t`).

Son implícitos ya que los argumentos de la función normalemente determinan sus valores. Sin embargo, no dejan de ser argumentos de la función ya que la misma depende de éstos.

###### Aplicación de parámetros implícitos

Este cuantificador es el único que puede recibir una aplicación implícita, ya que sirve _para todos_ los valores:

```haskell
<expr> { <iden> = <expr> }
```

Un ejemplo sería utilizarlo para reducir el dominio de una función:

```haskell
map {b = Bool} : forall a:Type . (a -> Bool) -> List a -> List Bool
```

#### Cuantificador _exists_

Para asociar identificadores a un valor específico de un tipo dado, como testigo.

```haskell
exists <iden> : <type> ; <expr>
```

Ésto introduce el identificador mencionado y con el tipo indicado, pero retorna una _tupla_ con el valor calculado y el valor encontrado para el existencial.

Un ejemplo es la función para filtrar elementos de un vector:

```haskell
vfilter : forall a:Type, n:Nat . (a -> Bool) -> Vect n a -> exists m:Nat ; Vect m a
vfilter _ VNil = VNil
vfilter g (x <::> xs) = if g x then <* S l ; x <::> xs' *> else <* l ; xs' *>
    where
        <* l ; xs' *> = vfilter g xs
```

Se utiliza la función recién definida en el siguiente ejemplo:

```haskell
vfilter even (2 <::> 1 <::> 4 <::> VNil) : exists m:Nat ; Vect m Nat
```
Que es igual a:

```haskell
<* 2 ; 2 <::> 4 <::> VNil *>
```

La estructura del tipo existencial será definida más adelante.

#### Cuantificador _select_

Este _cuantificador_ sirve para referirse al valor del tipo indicado más adelante.

```haskell
select <iden> : <type>
```

Un ejemplo es la función identidad con anotación de tipo, que llamaremos `the`:

```haskell
the : (select t:Type) -> t -> t
the _ x = x
```

Esta función recibe el tipo que espera explícitamente, a diferencia de `id`, que lo recibe implícitamente.

Otro ejemplo es la conversión de lista a vector:

```haskell
ListToVect : forall t:Type . (select lst : List t) -> Vect (length lst) t
ListToVect Nil = VNil
ListToVect (x :: xs) = x <::> ListToVect xs
```

## Operadores

Los operadores sirven para usar ciertas funciones de manera _natural_; se definen con una asociatividad y precedencia, el identificador indica dónde espera los argumentos usando el caracter piso (`_`).

```haskell
operator <iden> <fixity> <prec>
```

donde `<fixity>` es una de las siguientes opciones:

- `postfix`: Para operadores _postfijos_.
- `prefix`: Para operadores _prefijos_.
- `infixL`: Para operadores _infijos_ con asociatividad izquierda.
- `infixR`: Para operadores _infijos_ con asociatividad derecha.
- `infixN`: Para operadores _infijos_ sin asociatividad.
- `closed`: Para operadores _cerrados_, éstos no tienen `_` al lado izquiero ni derecho del identificador.

y `<prec>` es un número natural para indicar su precedencia, donde mientras mayor sea el valor, mayor precedencia tiene el operador. Si la asociatividad es `closed`, no se indica la precedencia.

Algunos ejemplos válidos:

```haskell
operator _!            postfix 20
operator -_            prefix  19
operator _^_           infixR  17
operator _+_           infixL  15
operator _-_           infixL  15
operator _==_          infixN  10
operator _/=_          infixN  10
operator if_then_else_ prefix  5
operator <_;_>         closed
```

Estos operadores harán que expresiones que cumplan esas reglas, se _transformen_ a aplicaciones de funciones.

Por ejemplo:

```haskell
if a == b then - - x else y       -- if_then_else_ (_==_ a b) (-_ (-_ x)) y
- x! + y ^ 2 == z               -- _==_ (_+_ (-_ (_! x)) (_^_ y 2)) z
```

Sólo se buscarán estos _patrones_ al definir un operador explícitamente, es decir, los identificadores (aunque contengan `_`) no serán consdierados operadores, por lo que no sufrirán ninguna transformación.

También se puede usar en la definición de funciones:

```haskell
if_then_else_ : forall t:Type . Bool -> t -> t -> t
if True  then x else _ = x      -- if_then_else_ True  x _ = x
if False then _ else y = y      -- if_then_else_ False _ y = y
```

## Tipos

En Angler podemos declarar dos diferentes _tipos_ de tipos; los _cerrados_ y los _abiertos_.

### Tipos cerrados

Son tipos cuya definición completa es conocida, se usa la palabra reservada `closed` para su definición. Se define con un identificador, un tipo, y sus posibles constructores.


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

Un ejemplo un poco más complejo es el de listas y el de vectores:

```haskell
closed List : Type -> Type with
    Nil : forall t:Type . List t
    _::_ : forall t:Type . t -> List t -> List t

closed Vect : Nat -> Type -> Type with
    VNil : forall t:Type . Vect Z t
    _<::>_ : forall t:Type, n:Nat . t -> Vect n t -> Vect (S n) t
```

El tipo `_->_` es un tipo cerrado embebido en el lenguaje.

También `exists` es un tipo cerrado embebido en el lenguaje, se define de la siguiente forma:

```haskell
closed Exists : (select a:Type) -> (a -> Type) -> Type with
    <*_;_*> : forall a:Type, P:a -> Type . (select x:a) -> P x -> Exists a P
```

### Tipos abiertos

Se comportan como los tipos cerrados, pero sirven para conjuntos cuya definición completa es desconocida (o no acotada).

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

El tipo `Type` es un tipo abierto embebido en el lenguaje. Cada tipo nuevo se considera una _reapertura_ de éste.

## Comportamientos

Sirven para obtener un _estilo_ de sobrecarga de funciones, esto es, usar un mismo identificador para diferentes funciones (con tipos diferentes).

### Definición

Se definen con un identificador, un _argumento_ con su tipo, las funciones que define y sus definiciones por defecto (en caso de existir).

Las definiciones por defecto son definiciones que son derivables a partir de otras funciones, aunque siempre se puede indicar una implementación específica por parte del usuario.

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

Otro ejemplo es el comportamientode las cosas que son _mapeables_:

```haskell
behaviour Functor on f : Type -> Type defines
    fmap : forall a:Type, b:Type . (a -> b) -> f a -> f b
```

### Instanciación

Para definir que un tipo cumple con un comportamiento.

```haskell
<iden> is <iden> with
    <iden> [ <arg> ..] = <expr>
    [ <iden> [ <arg> ..] = <expr> ..]
```

Se lee «_tipo_ es un _comportamiento_, con ...».

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

Estas instancias nos abilitan, gracias a las definiciones por defecto, también el uso de la función `_/=_` para `Nat` y `Bool`.

Pero podríamos dar nuestra propia implementación:

```haskell
Bool is Eq with
    True == True = True
    False == False = True
    _ == _ = False

    True /= False = True
    False /= True = True
    _ /= _ False
```

Y las instancias de `Functor` para `Maybe` y `List` serían:

```haskell
Maybe is Functor with
    fmap _ Nothing = Nothing
    fmap f (Just x) = Just (f x)

List is Functor with
    fmap _ Nil = Nil
    fmap f (x :: xs) = f x :: fmap f xs
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

#### Definición de alcances de comportamientos abiertos

La definción de alcances de comportamientos abiertos puede ser simplemente la definción de un identificador para éste, o puede incluir instancias iniciales.

```haskell
open scope <iden> [with

    <iden> is <iden> with
        <iden> [ <arg> ..] = <expr>
        [ <iden> [ <arg> ..] = <expr> ..]

    [ <iden> is <iden> with
        <iden> [ <arg> ..] = <expr>
        [ <iden> [ <arg> ..] = <expr> ..]
    ..]
  ]
```

En cualquier parte posterior del código, se puede _reabrir_ un alcance de comportamientos abierto y agregar instancias a éste.

```haskell
reopen scope <iden> with

    <iden> is <iden> with
        <iden> [ <arg> ..] = <expr>
        [ <iden> [ <arg> ..] = <expr> ..]

    [ <iden> is <iden> with
        <iden> [ <arg> ..] = <expr>
        [ <iden> [ <arg> ..] = <expr> ..]
    ..]

```

Por ejemplo, haremos alcances abiertos para las distintas instancias de `Semigroup` y `Monoid` que puede tener `Bool`.

```haskell
open scope Any

open scope All with
    Bool is Semigroup with
        _<+>_ = _&&_

reopen scope Any with
    Bool is Semigroup with
        _<+>_ = _||_
    Bool is Monoid with
        neutral = False

reopen scope All with
    Bool is Monoid with
        neutral = True
```

#### Definición de alcances de comportamientos cerrados

Se asigna un nombre al alcance y se definen todas sus instancias inmediatamente.

```haskell
closed scope <iden> with

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
closed scope All with
    Bool is Semigroup with
        _<+>_ = _&&_

    Bool is Monoid with
        neutral = True

closed scope Any with
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

-- (isNeutral at All) True at Any           -- False? True?
-- ((isNeutral at All) True at Any) at All  -- False? True?
```

## Modo de pruebas

___TO DO___

## Azúcar Sintáctica

Hay ciertas construcciones en el lenguaje a las que se les podría dar una sintaxis especial para facilitar su escritura, esto lo llamamos _azúcar sintáctica_.

### Cuantificador existencial

El cuantificador existencial esta definido en Angler, pero su presentación habitual se hace con azúcar sintáctica, aquí presentamos ésta y su transformación:

```haskell
closed Exists : (select a:Type) -> (a -> Type) -> Type with¬
    <*_;_*> : forall a:Type, P:a -> Type . (select x:a) -> P x -> Exists a P
```

Así se haría uso de su definición:

```haskell
vfilter : forall a:Type, n:Nat . (a -> Bool) -> Vect n a -> Exists Nat (\m -> Vect m a)
```

Mientras que la azúcar sintáctica introduce el identificador (i.e. `m`) automáticamente:

```haskell
vfilter : forall a:Type, n:Nat . (a -> Bool) -> Vect n a -> exists m:Nat ; Vect m a
```

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

list : List Nat
list = [ 0 :: 1 :: 2 ]
```

### Cadenas de caracteres

La versión _azucarada_ de cadenas de caracteres (`String`) serían todos los caracteres seguidos entre dos comillas dobles (`"`).

```haskell
String : Type
String = List Char
```

Por ejemplo, `"Text\n"` es _desazucarado_ a `'T' :: 'e' :: 'x' :: 't' :: '\n' :: Nil`.

### Notación `do`

La notación `do` se usa para _Applicative_, por ejemplo:

```haskell
do
    x <- act0
    act1 x
    y <- act2
    act3 y x
```

Se convierte en:

```haskell
act0 >>= \x -> act1 x >> act2 >>= \y -> act3 y x
```
