# Angler

Angler es un lenguaje de programación funcional con un sistema de tipos dependientes orientado a la enseñanza de las matemáticas discretas.

## Módulo

Un módulo es el programa dentro de **un** archivo de Angler. Puede incluir una sección de *exportación*, y también de *importación*, ambas opcionales.

Las *importaciones* pueden ser *cualificadas*, es decir, agregando a su identificador un prefijo.

~~~haskell
export ( [ <identificador> [ , <identificador> ..] ] )

import <identificador>  [ as <identificador> ] [ ( [identificador [ , <identificador> ..] ] ) ]
~~~

Por ejemplo:

~~~haskell
export (IntMap, member, lookup, insert, update, keys, elems)

import Maybe        (Maybe, Just, Nothing)      -- Maybe, Just, Nothing
import Maybe        (Maybe(Just, Nothing))      -- Maybe, Just, Nothing
import List  as L   (lookup, foldr)             -- L.lookup, L.fodlr
import Map   as Map                             -- Map.*
import Show                                     -- *
~~~

## Comentarios

Comentarios como en Haskell.

~~~haskell
-- Así son de una línea

{-
    Así son de varias líneas
-}
~~~

## Funciones

El símbolo `:` es reservado para el lenguaje, y el símbolo `->` es un operador que viene embebido.

~~~haskell
curry : forall a:Type, b:Type, c:Type . (a /\ b ->  c) -> a -> b -> c
curry f a b = f <a, b>
~~~

### Declaración y definición

Declarar una función es declarar un tipo, pero usar un identificador en letra minúscula, esto debería hacer que el compilador se *queje* si no hay una *definición* para esta función.

~~~haskell
<identificador> : <tipo> [ where <alcance> ]
~~~

Definir una función es explicar el comportamiento de esta por los argumentos *recibidos*, se usa la sintaxis:

~~~haskell
<identificador> [ <argumento> [ <argumento> ..] ] = <expresión> [ where <alcance> ]
~~~

Al final de una `<expresión>` o `<tipo>` se puede colocar la palabra reservada `where` para definir funciones dentro de un alcance sólo accesible por ésta expresión o tipo.

En el alcance del `where` sólo se pueden declarar funciones, definir funciones, declarar tipos cerrados y declarar operadores.

#### *Don't care*

Cuando no nos interesa usar un valor, en vez de darle un nombre poco significativo para hacerle referencia, le colocamos `_`, indicando que no haremos referencia a él más adelante.

~~~haskell
isUSD : Currency -> Bool
isUSD (USD _) = True
isUSD _        = False
~~~

#### λ-funciones

Funciones que no tienen un identificador.

~~~haskell
\x -> x * 2
~~~

Esta función es de tipo `Nat -> Nat`.

### Operadores

Un operador se define dándole una asociatividad y una precedencia, el identificador del operador debe indicar dónde van sus argumentos usando el caracter *piso* (`_`).

~~~haskell
operator _!  postfix 1
operator -_   prefix 2
operator _+_  infixL 3
operator _-_  infixL 3
operator _/\_ infixR 5
operator _==_ infixN 6
~~~

Luego se utiliza el mismo identificador

~~~haskell
_+_ : Nat -> Nat -> Nat
-- n + Z     = n                -- ¿capaz?
-- n + (S m) = S (n + m)        -- ¿capaz?
_+_ n Z     = n
_+_ n (S m) = S (n + m)

operator if_then_else_ prefix 7         

-- if-then-else definido en el lenguaje, 😱
if_then_else_ : forall t:Type . Bool -> t -> t -> t
if_then_else_ True  x _ = x
if_then_else_ False _ x = x
~~~

### Parámetros implícitos

Si la función tiene *polimorfismo de tipos*, se deben indicar con parámetros implícitos de qué tipos seran.

#### *forall*

Para indicar un parámetro implícito que aplica para cualquier elemento de ese tipo.

~~~haskell
forall <identificador> : <tipo> [ , <identificador> : <tipo> ..] .
~~~

Ejemplo:

~~~haskell
id : forall t:Type . t -> t
id x = x
~~~

#### *exists*

Para indicar un parámetro implícito que aplica para *algún* elemento de ese tipo.

~~~haskell
exists <identificador> : <tipo> [ , <identificador> : <tipo> ..] .
~~~

Ejemplo:

~~~haskell
filter : forall a:Type, n:Nat . (a -> Bool) -> Vect n a -> (exists m:Nat ; Vect m a)
filter _ Nil       = Nil
filter g (x :: xs) = (if g x then (\xs' -> x :: xs') else id) filter g xs
~~~

### Parámetros explícitos con reutilización (REVISAR TÍTULO)

Para recibir un valor interesante a utilizar más adelante, se usa el *cuantificador* `select`.

~~~haskell
the : (select t:Type) -> t -> t
the t x = x
~~~

### Llamadas de funciones

Se llama una función colocando su identificador y seguidamente valores a ser pasados como argumentos.

~~~haskell
<identificador> [ <expresión> [ <expresión> ..] ]
~~~

Ejemplo:

~~~haskell
the Nat (S Z)                   -- = S Z        : Nat

filter even (1::2::3::4::Nil)   -- = 2::4::Nil  : (2 ; Vect 2 Nat)

filter odd                      -- : forall n:Nat . Vect n Nat -> exists m:Nat ; Vect m Nat
~~~

#### Pasando argumentos implícitos

Se puede instanciar directamente los argumentos implícitos usando llaves (`{`, `}`). Probablemente únicamente del `forall`.

~~~haskell
<identificador> [ { <implícito> = <expresión> [ , <implícito> = <expresión> ] } ] [ <expresión> [ <expresión> ..] ]
~~~

Ejemplo:

~~~haskell
id {t = Nat} (S Z)

filter odd { n = 10 }   -- : Vect 10 Nat -> (exists m:Nat ; Vect m Nat)
~~~

### Ejemplos de funciones

~~~haskell
not : Bool -> Bool
not True  = False
not False = True

_&&_ : Bool -> Bool -> Bool
False && _ = False
True  && x = x

_||_ : Bool -> Bool -> Bool
True  || _ = True
False || x = x

id : forall t:Type . t -> t
id x = x

the : (select t:Type) -> t -> t
the _ x = x

const : forall a:Type, b:Type . a -> b -> a
const x _ = x

map : forall a:Type, b:Type . (a -> b) -> List a -> List b
map _ Nil     = Nil
map f (x::xs) = f x :: map f xs

-- llamadas de funciones

map (\x -> x * 2) [1,2,3]           : List Nat -- azúcar sintáctica
map (\x -> x * 2) (1::2::3::Nil)    : List Nat -- desazucarea a esto

map {a = Nat}                       : forall b:Type . (Nat -> b) -> List Nat -> List b

map {a = Nat, b = Bool}             : (Nat -> Bool) -> List Nat -> List Bool
map (const True)                    : forall a:Type . List a -> List Bool
~~~

## Sistema de tipos

### Declaración de tipos

El tipo `Type` viene embebido en el lenguaje, probablemente.

#### Tipos cerrados

Los *tipos cerrados* usan la palabra reservada `closed` e indican después de `with` sus constructores con indentación.

~~~haskell
closed Bool : Type with
    True : Bool
    False : Bool
~~~

#### Tipos abiertos

Los *tipos abiertos* se declaran usando la palabra reservada `open`, y pueden incluir la palabra reservada `with` para indicar constructores.

Luego de haber definido un tipo abierto, se puede reabrir con la palabra reservada `reopen` y colocarle constructores nuevos después de la palabra reservada `with`.

~~~haskell
open Currency : Type with
    USD : Nat -> Currency
    EUR : Nat -> Currency

-- ...
reopen Currency with
    VEF : Nat -> Currency

-- Otro ejemplo

open Option : Type

-- ...

reopen Option with
    One : Option
    Two : Option
~~~

#### Tipos dependientes

Tipos que dependen de otros valores para ser construídos.

~~~haskell
closed List : (select a:Type) -> Type with
    Nil  : List a
    _::_ : a -> List a -> List a
~~~

#### Aliases

Definir un *alias* para un tipo que se escriba mucho, o para darle mayor significado a lo que estamos haciendo.

~~~haskell
String : Type
String = List Char
~~~

### Otras cosas de tipos...

### Ejemplos de tipos

~~~haskell
closed Bool : Type with
    True : Bool
    False : Bool

closed Nat : Type with
    Z : Nat
    S : Nat -> Nat

-- And / Tuples
closed _/\_ : (select a:Type) -> (select b:Type) -> Type with
    <_,_> : a -> b -> a /\ b

-- Should this type be in the standard?
closed _\/_ : (select a:Type) -> (select b:Type) -> Type with
    _<|   : a -> a \/ b
      |>_ : b -> a \/ b
    _<|>_ : a -> b -> a \/ b

-- XOR
closed _=/=_ : (select a:Type) -> (select b:Type) -> Type with
    _</=  : a -> a =/= b
     =/>_ : b -> a =/= b
closed Either : (select a:Type) -> (select b:Type) -> Type with
    Left  : a -> Either a b
    Right : b -> Either a b

closed Maybe : (select a:Type) -> Type with
    Nothing : Maybe a
    Just    : a -> Maybe a
closed Maybe' : Type -> Type with
    Nothing : forall (a:Type) . Maybe' a
    Just    : forall (a:Type) . a -> Maybe' a

closed List : (select a:Type) -> Type with
    Nil  : List a
    _::_ : a -> List a -> List a
closed List' : Type -> Type with
    Nil  : forall (a:Type) . List' a
    _::_ : forall (a:Type) . a -> List' a -> List' a

closed Tree : (select a:Type) -> Type with
    Leaf   : a -> Tree a
    Branch : Tree a -> Tree a -> Tree a

closed Vect : Nat -> (select a:Type) -> Type with
    Nil  : Vect Z a
    _::_ : forall n:Nat . a -> Vect n a -> Vect (S n) a
~~~

## Comportamientos

### Definición

Son el equivalente a *clases* de Haskell. La sintaxis es

~~~haskell
<comportamiento> on <identificador> : <tipo> where
    <declaración función>
    [ <declaración función> ..]
[ with
    <definición función>
    [ <definición función> ..] ]
~~~

Por ejemplo:

~~~haskell
Eq on t : Type where
    _==_ : t -> t -> Bool
    _/=_ : t -> t -> Bool
with
    a == b = not (a /= b)
    a /= b = not (a == b)
~~~

### Instanciación

Se instancia con la siguiente sintaxis

~~~haskell
<identificador de tipo> is <comportamiento> where
    <definición función>
    [ <definición función> ..]
~~~

Deben definirse el mínimo para que se definan todas con el `with` del comportamiento.

Por ejemplo:

~~~haskell
Nat is Eq where
    Z     == Z     = True
    (S n) == (S m) = n == m
    _     == _     = False
~~~

### ¿Petición?

Se pueden pedir que un tipo tenga un cierto comportamiento, aún no sabemos cómo.

~~~haskell
<comportamiento> <identificador> : <tipo>

<identificador> is <comportamiento> : <tipo>

(<identificador> : <tipo>) is <comportamiento>
~~~

Por ejemplo:

~~~haskell
contains : forall    Eq t : Type . t -> List t -> Bool
contains : forall t is Eq : Type . t -> List t -> Bool
contains : forall (t:Type) is Eq . t -> List t -> Bool

contains _ Nil     = False
contains a (x::xs) = (a == x) || contains a xs        -- cortocircuito


Semigroup on t : Type where
    _<+>_ : t -> t -> t
Monoid on t is Semogroup : Type where
    neutral : t
~~~

### ¿Alcance? de comportamientos

#### ¿Alcances? abiertos

~~~haskell
behavespace <alcance>

<identificador de tipo> is <comportamiento> at <alcance> where
    <definición función>
    [ <definición función> ..]

...

<identificador de tipo> is <comportamiento> at <alcance> where
    <definición función>
    [ <definición función> ..]
~~~

#### ¿Alcances? cerrados

~~~haskell
behavespace <alcance> where

    <identificador de tipo> is <comportamiento> where
        <definición función>
        [ <definición función> ..]

    <identificador de tipo> is <comportamiento> where
        <definición función>
        [ <definición función> ..]
~~~

Por ejemplo:

~~~haskell
behavespace All
Bool is Semigroup at All where
    _<+>_ = _&&_
Bool is Monoid at All where
    neutral = True

behavespace Any where
    Bool is Semigroup where
        _<+>_ = _||_
    Bool is Monoid where
        neutral = False
~~~

#### Alcance en pasaje de parámetros

~~~haskell
monoId : forall a is Monoid : Type . a -> a
monoId x = neutral <+> x

monoId True                     -- Error, no Monoid
monoId {a = Bool@All} True      -- True  && True
monoId {a = Bool@Any} True      -- False || True
~~~

### Ejemplos de comportamientos

~~~haskell
Eq on t : Type where
    _==_ : t -> t -> Bool
    _/=_ : t -> t -> Bool
with
    a == b = not (a /= b)
    a /= b = not (a == b)

Nat is Eq where
    Z     == Z     = True
    (S n) == (S m) = n == m
    _     == _     = False
~~~

## Modo de pruebas

***TODO***

## Azúcar Sintáctica

Hay ciertas partes de la sintaxis del lenguaje que podríamos facilitar para la escritura, esto es llamado *azúcar sintáctica* (syntactic sugar).

Aquí se listaran **posibles** casos de azúcar sintáctica.

### Listas

La versión *azucarada* de listas sería `[0, 1, 2]`, para ser *desazucarado* a `0 :: 1 :: 2 :: Nil`.

### Cadenas de caracteres

La versión *azucarada* de cadenas de caracteres (`String`) sería `"Text\n"`, para ser *desazucarado* a `'T' :: 'e' :: 'x' :: 't' :: '\n' :: Nil`.

### Notación `do`

La notación `do` se usa para *monads*:

~~~haskell
do
    x <- action0
    action2 x
    z <- action3
    action4 z
~~~

Se convierte en:

~~~haskell
action0 >>= \x =>
    action2 x >>
        action3 >>= \z =>
            action4 z  
~~~
