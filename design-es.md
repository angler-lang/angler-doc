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

## Identificadores

Los identificadores en Angler tienen las siguientes reglas de construcción:

1. Los identificadores constan de una o más _partes_
2. Las _partes_ puden ser de las siguientes formas:
    1. Comienzan por una letra, seguida de cero o más letras, números o comillas simples. -- `alpha [alpha digit ']*`
    2. Comienzan por un símbolo, seguido de cero o más símbolo o comillas simples. -- `symbol [symbol ']*`

        _Símbolos_: ``- ! @ # $ % & * + / \ < = > ^ | ~ ? ` [ ] : ; .``

3. Entra partes debe haber un caracter *piso* (`_`)
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
- `_`: Para denotar un valor que no es de interés (don't care)
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

Un módulo es el programa dentro de **un** archivo de Angler. Puede incluir una sección de _exportación_, y también de _importación_, ambas opcionales.

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

Los comentarios en Angler son como en Haskell. Los comentarios pueden comenzar en cualquier punto del programa.

```haskell
{-
    Function _+_
    Addition for Nat (Natural numbers)
-}
_+_ : Nat -> Nat -> Nat
Z + m = m                   -- base case of addition
S n + m = S (n + m)         -- recursive case
```

## Funciones

Una función consta de dos partes, su _declaración_ y su(s) _definición(es)_, la definición es opcional.

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

En el ejemplo de la función identidad, la declaración es la siguiente línea `id : forall t:Type . t -> t`. La función de composición puede tener la siguiente declaración:

```haskell
_._ : forall a:Type, b:Type, c:Type . (b -> c) -> (a -> b) -> (a -> c)
```

Nótese que el operador `->` es asociativo a la derecha, por lo que la siguiente declaración es equivalente:

```haskell
_._ : forall a:Type, b:Type, c:Type . (b -> c) -> (a -> b) -> a -> c
```

Esto declara el tipo que debe tener la función a ser definida.

### Definición de funciones

Una definición de función consta de su identificador, seguido por sus argumentos, y la expresión que la *define*, separados por `=`.

```haskell
<iden> [<iden> ..] = <expr>
```

En el ejemplo de la función identidad, la definición es la línea `id x = x`; donde se separa en la siguientes partes:

- __`id x`__` = x`: el identificador (`id`) y su único argumento `x`.
- `id x`__` = `__`x`: el `=` que separa la expresión que la define de sus argumentos.
- `id x = `__`x`__: la expresión que define a la función.

La función de composición puede tener la siguente definición:

```haskell
_._ f g x = f (g x)
```

Aquí tenemos tres el identificador `_._`, los argumentos `f`, `g` y `x`, y la expresión `f (g x)`.

Si definimos un operador para `_._` (`operator _._ infixR 9`), podríamos definir la función composición de la siguiente forma:

```haskell
(f . g) x = f (g x)
```

Que es *interesante*, pues así se usaría en código probablemente.

****
****
****
****
****
****
****
****
****
****
****
****
****
****
****
****
****
****
****
****
****
****

### Declaración y definición

Declarar una función es declarar un tipo, pero usar un identificador en letra minúscula, esto debería hacer que el compilador se _queje_ si no hay una _definición_ para esta función.

```haskell
<identificador> : <tipo> [ where <alcance> ]
```

Definir una función es explicar el comportamiento de esta por los argumentos _recibidos_, se usa la sintaxis:

```haskell
<identificador> [ <argumento> [ <argumento> ..] ] = <expresión> [ where <alcance> ]
```

Al final de una `<expresión>` o `<tipo>` se puede colocar la palabra reservada `where` para definir funciones dentro de un alcance sólo accesible por ésta expresión o tipo.

En el alcance del `where` sólo se pueden declarar funciones, definir funciones, declarar tipos cerrados y declarar operadores.

#### *Don't care*

Cuando no nos interesa usar un valor, en vez de darle un nombre poco significativo para hacerle referencia, le colocamos `_`, indicando que no haremos referencia a él más adelante.

```haskell
isUSD : Currency -> Bool
isUSD (USD _) = True
isUSD _        = False
```

#### λ-funciones

Funciones que no tienen un identificador.

```haskell
\x -> x * 2
```

Esta función es de tipo `Nat -> Nat`.

### Operadores

Un operador se define dándole una asociatividad y una precedencia, el identificador del operador debe indicar dónde van sus argumentos usando el caracter _piso_ (`_`).

```haskell
operator _!  postfix 1
operator -_   prefix 2
operator _+_  infixL 3
operator _-_  infixL 3
operator _/\_ infixR 5
operator _==_ infixN 6
```

Luego se utiliza el mismo identificador

```haskell
_+_ : Nat -> Nat -> Nat
-- n + Z     = n                -- ¿capaz?
-- n + (S m) = S (n + m)        -- ¿capaz?
_+_ n Z     = n
_+_ n (S m) = S (n + m)

operator if_then_else_ prefix 7         

-- if-then-else definido en el lenguaje, ��
if_then_else_ : forall t:Type . Bool -> t -> t -> t
if_then_else_ True  x _ = x
if_then_else_ False _ x = x
```

### Parámetros implícitos

Si la función tiene _polimorfismo de tipos_, se deben indicar con parámetros implícitos de qué tipos seran.

#### Aplicación de parámetros implísitos


```haskell
map : forall a:Type, b:Type, n:Nat . (a -> b) -> Vect n a -> Vect n b
map {a = Nat, b = Char}
map {a = Nat} {b = Char}

map : forall a:Type . forall b:Type . (a -> b) -> forall n:Nat . Vect n a -> Vect n b
map {a = Nat} {b = Char}
```

#### *forall*

Para indicar un parámetro implícito que aplica para cualquier elemento de ese tipo.

```haskell
forall <identificador> : <tipo> [ , <identificador> : <tipo> ..] .
```

Ejemplo:

```haskell
id : forall t:Type . t -> t
id x = x
```

#### *exists*

Para indicar un parámetro implícito que aplica para _algún_ elemento de ese tipo.

```haskell
exists <identificador> : <tipo> [ , <identificador> : <tipo> ..] .
```

Ejemplo:

```haskell
filter : forall a:Type, n:Nat . (a -> Bool) -> Vect n a -> (exists m:Nat ; Vect m a)
filter _ Nil       = Nil
filter g (x :: xs) = (if g x then (\xs' -> x :: xs') else id) filter g xs
```

### Parámetros explícitos con reutilización (REVISAR TÍTULO)

Para recibir un valor interesante a utilizar más adelante, se usa el _cuantificador_ `select`.

```haskell
the : (select t:Type) -> t -> t
the t x = x
```

### Llamadas de funciones

Se llama una función colocando su identificador y seguidamente valores a ser pasados como argumentos.

```haskell
<identificador> [ <expresión> [ <expresión> ..] ]
```

Ejemplo:

```haskell
the Nat (S Z)                   -- = S Z        : Nat

filter even (1::2::3::4::Nil)   -- = 2::4::Nil  : (2 ; Vect 2 Nat)

filter odd                      -- : forall n:Nat . Vect n Nat -> exists m:Nat ; Vect m Nat
```

#### Pasando argumentos implícitos

Se puede instanciar directamente los argumentos implícitos usando llaves (`{`, `}`). Probablemente únicamente del `forall`.

```haskell
<identificador> [ { <implícito> = <expresión> [ , <implícito> = <expresión> ] } ] [ <expresión> [ <expresión> ..] ]
```

Ejemplo:

```haskell
id {t = Nat} (S Z)

filter odd { n = 10 }   -- : Vect 10 Nat -> (exists m:Nat ; Vect m Nat)
```

### Ejemplos de funciones


```haskell
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
```

## Sistema de tipos

### Declaración de tipos

El tipo `Type` viene embebido en el lenguaje, probablemente.

#### Tipos cerrados

Los _tipos cerrados_ usan la palabra reservada `closed` e indican después de `with` sus constructores con indentación.

```haskell
closed Bool : Type with
    True : Bool
    False : Bool
```

#### Tipos abiertos

Los _tipos abiertos_ se declaran usando la palabra reservada `open`, y pueden incluir la palabra reservada `with` para indicar constructores.

Luego de haber definido un tipo abierto, se puede reabrir con la palabra reservada `reopen` y colocarle constructores nuevos después de la palabra reservada `with`.

```haskell
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
```

#### Tipos dependientes

Tipos que dependen de otros valores para ser construídos.

```haskell
closed List : (select a:Type) -> Type with
    Nil  : List a
    _::_ : a -> List a -> List a
```

#### Aliases

Definir un _alias_ para un tipo que se escriba mucho, o para darle mayor significado a lo que estamos haciendo.

```haskell
String : Type
String = List Char
```

### Otras cosas de tipos...

### Ejemplos de tipos


```haskell
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
```

## Comportamientos

### Definición

Son el equivalente a _clases_ de Haskell. La sintaxis es

```haskell
<comportamiento> on <identificador> : <tipo> where
    <declaración función>
    [ <declaración función> ..]
[ with
    <definición función>
    [ <definición función> ..] ]
```

Por ejemplo:

```haskell
Eq on t : Type where
    _==_ : t -> t -> Bool
    _/=_ : t -> t -> Bool
with
    a == b = not (a /= b)
    a /= b = not (a == b)
```

### Instanciación

Se instancia con la siguiente sintaxis

```haskell
<identificador de tipo> is <comportamiento> where
    <definición función>
    [ <definición función> ..]
```

Deben definirse el mínimo para que se definan todas con el `with` del comportamiento.

Por ejemplo:

```haskell
Nat is Eq where
    Z     == Z     = True
    (S n) == (S m) = n == m
    _     == _     = False
```

### ¿Petición?

Se pueden pedir que un tipo tenga un cierto comportamiento, aún no sabemos cómo.

```haskell
<comportamiento> <identificador> : <tipo>

<identificador> is <comportamiento> : <tipo>

(<identificador> : <tipo>) is <comportamiento>
```

Por ejemplo:

```haskell
contains : forall    Eq t : Type . t -> List t -> Bool
contains : forall t is Eq : Type . t -> List t -> Bool
contains : forall (t:Type) is Eq . t -> List t -> Bool

contains _ Nil     = False
contains a (x::xs) = (a == x) || contains a xs        -- cortocircuito


Semigroup on t : Type where
    _<+>_ : t -> t -> t
Monoid on t is Semogroup : Type where
    neutral : t
```

### ¿Alcance? de comportamientos

#### ¿Alcances? abiertos


```haskell
behavespace <alcance>

<identificador de tipo> is <comportamiento> at <alcance> where
    <definición función>
    [ <definición función> ..]

...

<identificador de tipo> is <comportamiento> at <alcance> where
    <definición función>
    [ <definición función> ..]
```

#### ¿Alcances? cerrados


```haskell
behavespace <alcance> where

    <identificador de tipo> is <comportamiento> where
        <definición función>
        [ <definición función> ..]

    <identificador de tipo> is <comportamiento> where
        <definición función>
        [ <definición función> ..]
```

Por ejemplo:

```haskell
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
```

#### Alcance en pasaje de parámetros


```haskell
monoId : forall a is Monoid : Type . a -> a
monoId x = neutral <+> x

monoId True                     -- Error, no Monoid
monoId {a = Bool@All} True      -- True  && True
monoId {a = Bool@Any} True      -- False || True
```

### Ejemplos de comportamientos


```haskell
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
```

## Modo de pruebas

**_TODO_**

## Azúcar Sintáctica

Hay ciertas partes de la sintaxis del lenguaje que podríamos facilitar para la escritura, esto es llamado _azúcar sintáctica_ (syntactic sugar).

Aquí se listaran **posibles** casos de azúcar sintáctica.

### Listas

La versión _azucarada_ de listas sería `[0, 1, 2]`, para ser _desazucarado_ a `0 :: 1 :: 2 :: Nil`.

### Cadenas de caracteres

La versión _azucarada_ de cadenas de caracteres (`String`) sería `"Text\n"`, para ser _desazucarado_ a `'T' :: 'e' :: 'x' :: 't' :: '\n' :: Nil`.

### Notación `do`

La notación `do` se usa para _monads_:

```haskell
do
    x <- action0
    action2 x
    z <- action3
    action4 z
```

Se convierte en:

```haskell
action0 >>= \x =>
    action2 x >>
        action3 >>= \z =>
            action4 z
```
