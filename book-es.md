# Introducción

# Marco Teórico

# Marco Tecnológico

# Marco Metodológico

# Desarrollo

Qué sucedió para llegar al resultado final.

## Investigación

## Diseño e implementación

**NOTA**: Se usarán algunos ejemplos de tipos básicos para hacer obvia la evolución.

Comenzamos con algunas ideas básicas de qué buscabamos en el diseño de este lenguaje:

- con una definción compacta
- explícito
- para principiantes
- con tipos dependientes

### Expresiones, tipos y sus construcciones

Ya que el lenguaje debe tener tipos dependientes, se decidió desde un principio que un tipo es sencillamente una expresión, no un elemento especial del lenguaje, hace que en cualquier parte donde se espera un tipo, debería poder escribirse cualquier expresión, mientras que el chequeo de tipos pase sin errores.

Al hacer estos dos elementos uno solo, se definieron las diferentes construcciones posibles de expresiones.

- Aplicación de funciones
- Parentización
- Lambda funciones
- Pattern matching
- Alcance de declaraciones para una expresión (`where`, `let-in`)
- Cuantificadores universales y existenciales de tipos

### Tipos y funciones, ¿cerrados o abiertos?

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
        True : b where b = Bool
        False : b where b = Bool
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

### Identificadores, ¿mayúsculas o minúsculas?

En un principio se consideró usar la convención de Haskell, que los constructores de tipos y tipos se escribieran con la primera letra mayúscula, y que los valores con la primera letra minúscula; de esta manera se podía distinguir sintácticamente entre ellos.

Pero esta idea tenía algunos problemas, por ejemplo, los identificadores que comenzaran con símbolos y no letras, ¿a qué clase pertenecerían?, además debemos recordar que con tipos dependientes, los tipos son expresiones como cualquier otra, así que la distinción entre tipos y valores no es lógica en este caso.

### Cuantificadores: parámetros y argumentos implícitos

Como se busca un lenguaje explícito, todo identificador que se utilice en una expresión, debe estar previamente definido con un tipo acompañante, tomando inspiración de otros lenguajes con tipos dependientes llegamos a la idea de tener dos secciones de la firma de una función, la parte _implícita_, que es el cuantificador universal; y la parte _explícita_:

```haskell
const : { (a : Type) -> (b : Type) } -> a -> b -> a
```

Donde se escribe la parte implícita entre llaves (`{` y `}`) como un tipo más, y luego se escribe la parte explícita, donde todos los nombres introducidos en la parte implícita están disponibles. Esto nos dejaba con dudas acerca de cómo se podían escribir con una estructura similar los tipos existenciales.

Los argumentos implícitos se pasan a la función al comenzar su nombre con un caracter _piso_ (`_`):

```haskell
const _Bool : { (b : Type) } -> Bool -> b -> Bool
```

Esta manera de pasar argumentos implícitos nos da una restricción en los identificadores que se puedene escribir en el lenguaje, ya que ninguno podría comenzar con piso para poder diferenciarlos de argumentos implícitos.

Pero hay un problema con la propuesta anterior, recordemos que en la teoría de tipos dependientes tenemos cuantificadores universales y existenciales; en un cuantificador universal no importa el orden de las variables declaradas, se puede especificar un valor para cualquera de las variables disponibles, es decir, los parámetros implícitos deberían ser todos accesibles para el pasaje de argumentos implícitos. Así llegamos a otra propuesta que separa completamente las firmas de los cuantificadores:

```haskell
const : forall a:Type, b:Type . a -> b -> a
```

Ahora la definición se asemeja mucho más a un cuantificador, haciendo más explícito su uso, y además podemos acceder a todas las variables declaradas y especificarlas de la siguiente forma:

```haskell
const { b = Char } : forall a:Type . a -> Char -> a
```

Donde accedimos a la segunda variable declarada. Como pueden ver, no abandonamos por completo la idea de usar llaves (`{` y `}`) para parámetros implícitos, ahora se usa para los argumentos.

Se consideró usar la palabra `product` en vez de `forall`, pero se decidió usar `forall` para que hubiera una referencia directa a lógica simbólica, aunque estos se consideren los tipos producto.

De igual forma, se definen los tipos existenciales con la palabra `exists`, aunque se consideró la palabra `sum`:

```haskell
vfilter : forall t:Type, n:Nat . (t -> Bool) -> Vect n t -> exists m:Nat . Vect m t
```

Logrando una estructura parecida entre los cuantificadores universales y existenciales.

### Comportamientos

## Implementación

### Etapas
#### Lexer
#### Parser
#### Mixfix Parser
#### Type-checking

***********

## Cuando ...

***********

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
