# Angler

Angler es un lenguaje de programación funcional con un sistema de tipos dependientes orientado a la enseñanza de las matemáticas discretas.

## Comentarios

Comentarios como en Haskell.

    -- Así son de una línea.

    {-
        Así son de varias líneas.
     -}

## Funciones

El operador `:` viene embebido en el lenguaje, si se considera un operador. ¿Y el operador `->`?.

### Declaración y definición

Declarar una función es declarar un tipo, pero usar un identificador en letra minúscula, esto debería hacer que el compilador se *queje* si no hay una *definición* para esta función.

    <identificador> : <tipo>

Definir una función es explicar el comportamiento de esta por los argumentos *recibidos*, se usa la sintáxis:

    <identificador> [<argumento> [<argumento>..]] = <expresión>

#### Funciones abiertas

Las *funciones abiertas* van de la mano con los *tipos abiertos*, se pueden definir más casos cuando se vayan necesitando.

    toUSD : Currency -> Nat
    toUSD (USD c) = c
    toUSD (EUR c) = c * 2
    -- ...

#### Funciones cerradas

Las *funciones cerradas* usan la palabra reservada `where` para indicar con indentación sus casos a considerar.

    isUSD : Currency -> Bool where
        isUSD (USD ?) = True
        isUSD ?       = False

#### *Don't care*

Cuando no nos interesa usar un valor, en vez de darle un nombre poco significativo para hacerle referencia, le colocamos `?`, indicando que no haremos referencia a él más adelante.

    isUSD : Currency -> Bool where
        isUSD (USD ?) = True
        isUSD ?       = False

#### λ-funciones

Funciones que no tienen un identificador.

    \x => x * 2

Ésta función es de tipo `Nat -> Nat` (o con algún número, definir bien).

### Operadores

Tienen una sintáxis especial para lograr una declaración intuitiva. Se usa el caracter *piso* (`_`) para indicar dónde van los argumentos.

    _+_ : Nat -> Nat -> Nat
    n + Z     = n
    n + (S m) = S (n + m)

    -- if-then-else definido en el lenguaje, 😱
    if_then_else_ : forall t:Type . Bool -> t -> t -> t
    if True  then happens else ?       = happens
    if False then ?       else happens = happens

### Parámetros implícitos

Si la función tiene *polimorfismo de tipos*, se deben indicar con parámetros implícitos de qué tipos seran.

#### *forall*

Para indicar un parámetro implícito que aplica para cualquier elemento de ese tipo.

    forall <identificador> : <tipo> [, <identificador> : <tipo>...] .

Ejemplo:

    id : forall t:Type . t -> t
    id x = x

#### *exists*

Para indicar un parámetro implícito que aplica para *algún* elemento de ese tipo.

    exists <identificador> : <tipo> [, <identificador> : <tipo>...] .

Ejemplo:

    filter : forall a:Type, n:Nat . (a -> Bool) -> Vect n a -> exists m:Nat . Vect m a
    filter ? Nil       = Nil
    filter g (x :: xs) = (if g x then \xs' => x :: xs' else id) filter g xs

### Parámetros explícitos con reutilización (REVISAR TÍTULO)

Para recibir un valor interesante a utilizar más adelante, se usa el *cuantificador* `with`.

    the : (with t:Type) -> t -> t
    the t x = x

### Llamadas de funciones

Se llama una función colocando su identificador y seguidamente valores a ser pasados como argumentos.

    <identificador> [<expresión> [<expresión>..]]

Ejemplo:

    the Nat (S Z)

Esa llamada da igual a `S Z`.

#### Pasando argumentos implícitos

Se puede instanciar directamente los argumentos implícitos usando llaves (`{`, `}`). Probablemente únicamente del `forall`.

    <identificador> [{ <implícito> = <expresión>[, <implícito> = <expresión> ]}] [<expresión> [<expresión>..]]

Ejemplo:

    id {t = Nat} (S Z)

Esa llamada da igual a `S Z`.

### Ejemplos de funciones

    not : Bool -> Bool
    not True  = False
    not False = True

    _&&_ : Bool -> Bool -> Bool
    False && ? = False
    True  && x = x

    _||_ : Bool -> Bool -> Bool
    True  || ? = True
    False || x = x

    id : forall t:Type . t -> t
    id x = x

    the : (with t:Type) -> t -> t
    the ? x = x

    const : forall a:Type, b:Type . a -> b -> a
    const x ? = x

    map : forall a:Type, b:Type . (a -> b) -> List a -> List b
    map ? Nil     = Nil
    map f (x::xs) = f x :: map f xs

    map (\x => x * 2) [1,2,3]           : List Nat -- azúcar sintáctica
    map (\x => x * 2) (1::2::3::Nil)    : List Nat -- desazucarea a esto

    map {a = Nat}                       : forall b:Type . (Nat -> b) -> List Nat -> List b

    map {a = Nat, b = Bool}             : (Nat -> Bool) -> List Nat -> List Bool
    map (const True)                    : forall a:Type . List a -> List Bool


## Sistema de tipos

### Declaración de tipos

El tipo `Type` viene embebido en el lenguaje, probablemente.

#### Tipos cerrados

Los *tipos cerrados* usan la palabra reservada `where` para indicar con indentación sus constructores.

    Bool : Type where
        True : Bool
        False : Bool

#### Tipos abiertos

Los *tipos abiertos* no utilizan la palabra reservada `where`, sino que se van escribiendo constructores cuando se vayan necesitando.

    Currency : Type

    USD : Nat -> Currency
    EUR : Nat -> Currency
    VEF : Nat -> Currency
    -- ...

#### Tipos dependientes

Tipos que dependen de otros valores para ser construídos.

    List : (with a:Type) -> Type where
        Nil  : List a
        _::_ : a -> List a -> List a

#### Aliases

Definir un *alias* para un tipo que se escriba mucho, o para darle mayor significado a lo que estamos haciendo.

    String : Type where
        String = List Char


### λ-tipos

???

### Otras cosas de tipos...

### Ejemplos de tipos

    Bool : Type where
        True : Bool
        False : Bool

    Nat : Type where
        Z : Nat
        S : Nat -> Nat

    -- And / Tuples
    _/\_ : (with a:Type) -> (with b:Type) -> Type where
        (_,_) : a -> b -> a /\ b

    -- Should this type be in the standard?
    _\/_ : (with a:Type) -> (with b:Type) -> Type where
        _<|   : a -> a \/ b
        |>_   : b -> a \/ b
        _<|>_ : a -> b -> a \/ b

    -- XOR
    _=/=_ : (with a:Type) -> (with b:Type) -> Type where
        _</= : a -> a =/= b
        =/>_ : b -> a =/= b
    Either : (with a:Type) -> (with b:Type) -> Type where
        Left  : a -> Either a b
        Right : b -> Either a b
    _|_ : (with a:Type) -> (with b:Type) -> Type where
        _<| : a -> a|b
        |>_ : b -> a|b

    Maybe : (with a:Type) -> Type where
        Nothing : Maybe a
        Just    : a -> Maybe a
    Maybe' : Type -> Type where
        Nothing : forall (a:Type) . Maybe' a
        Just    : forall (a:Type) . a -> Maybe' a

    List : (with a:Type) -> Type where
        Nil  : List a
        _::_ : a -> List a -> List a
    List' : Type -> Type where
        Nil  : forall (a:Type) . List' a
        _::_ : forall (a:Type) . a -> List' a -> List' a

    Tree : (with a:Type) -> Type where
        Leaf   : a -> Tree a
        Branch : Tree a -> Tree a -> Tree a

    Vect : Nat -> (with a:Type) -> Type where
        Nil  : Vect Z a
        _::_ : forall n:Nat . a -> Vect n a -> Vect (S n) a

## Comportamientos

### Definición

Son el equivalente a *clases* de Haskell. La sintaxis es

    <comportamiento> on <identificador> : <tipo> where
        <declaración función>
        [ <declaración función> ..]
    [ with
        <definición función>
        [ <definición función> ..] ]

Por ejemplo:

    Eq on t : Type where
        _==_ : t -> t -> Bool
        _/=_ : t -> t -> Bool
    with
        a == b = not (a /= b)
        a /= b = not (a == b)

### Instanciación

Se instancia con la siguiente sintaxis

    <identificador de tipo> is <comportamiento> where
        <definición función>
        [ <definición función> ..]

Deben definirse el mínimo para que se definan todas con el `with` del comportamiento.

Por ejemplo:

    Nat is Eq where
        Z     == Z     = True
        (S n) == (S m) = n == m
        ?     == ?     = False

### ¿Petición?

Se pueden pedir que un tipo tenga un cierto comportamiento, aún no sabemos cómo.

    <comportamiento> <identificador> : <tipo>

    <identificador> is <comportamiento> : <tipo>

    (<identificador> : <tipo>) is <comportamiento>

Por ejemplo:

    contains : forall    Eq t : Type . t -> List t -> Bool
    contains : forall t is Eq : Type . t -> List t -> Bool
    contains : forall (t:Type) is Eq . t -> List t -> Bool

    contains ? Nil     = False
    contains a (x::xs) = (a == x) || contains a xs        -- cortocircuito


    Semigroup on t : Type where
        _<+>_ : t -> t -> t
    Monoid on t is Semogroup : Type where
        neutral : t

### ¿Alcance? de comportamientos


#### ¿Alcances? abiertos

    behavespace <alcance>

    <identificador de tipo> is <comportamiento> at <alcance> where
        <definición función>
        [ <definición función> ..]

    ...

    <identificador de tipo> is <comportamiento> at <alcance> where
        <definición función>
        [ <definición función> ..]



#### ¿Alcances? cerrados

    behavespace <alcance> where

        <identificador de tipo> is <comportamiento> where
            <definición función>
            [ <definición función> ..]

        <identificador de tipo> is <comportamiento> where
            <definición función>
            [ <definición función> ..]


Por ejemplo:

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

#### Alcance en pasaje de parámetros

    monoId : forall a is Monoid : Type . a -> a
    monoId x = neutral <+> x

    monoId True                     -- Error, no Monoid
    monoId {a = Bool@All} True      -- True  && True
    monoId {a = Bool@Any} True      -- False || True

### Ejemplos de comportamientos

    Eq on t : Type where
        _==_ : t -> t -> Bool
        _/=_ : t -> t -> Bool
    with
        a == b = not (a /= b)
        a /= b = not (a == b)

    Nat is Eq where
        Z     == Z     = True
        (S n) == (S m) = n == m
        ?     == ?     = False

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

    do 
        x <- action0
        action2 x
        z <- action3
        action4 z

Se convierte en:

    action0 >>= \x => 
        action2 x >> 
            action3 >>= \z => 
                action4 z  
