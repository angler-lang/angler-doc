# Introducción

# Marco Teórico

# Marco Tecnológico

# Marco Metodológico

# Desarrollo

Qué sucedió para llegar al resultado final.

## Investigación

## Diseño

**NOTA**: Se usarán algunos ejemplos de tipos básicos para hacer obvia la evolución.

Comenzamos con algunas ideas básicas de qué buscabamos en el diseño de este lenguaje:

- con una definción compacta
- explícito
- para principiantes

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
Bool : Type  where
    True : Bool
    False : Bool

not : Bool -> Bool
not True = False
not False = True
```

Y ahora ninguno de las confusiones anteriores suedería, al declarar el `IDK` no habría problema, pero al llamar `not IDK`, el intepretador reportaría un error estático.

Pero parecía interensante la idea de tipos que sus constructores pudieran ser agregados mientras se fueran necesitando, viene en mente un ejemplo de monedas en el mundo con algunas monedas _base_, para que el usuario agregue las de interés.

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

Los tipos, abiertos y cerrados, deben indicar explícitamente cuáles son sus constructores de tipos para evitar el problema antes mencionado, de forma que lleagamos a la siguiente sintaxis:

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

Y dejamos de un lado la idea de funciones cerradas.

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
