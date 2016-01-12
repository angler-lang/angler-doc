# Consideraciones futuras

Este archivo almacenará ideas que se nos ocurran y capaz valga la pena implementar en un futuro.

## Diseño
- case .. of ..
- Typed holes
- Prueba de totalidad

## Implementación

- Cambiar todas las instancias de `Show` a Haskell válido, usar `PrettyShow` en cambio
- `_->_` es un constructor de `Type`
- `\ arg -> expr` es un constructor de `_->_`
- Agregar una clase `Expr` para que el mixfix parser funcione sobre argumentos y expresiones pre y post procesamiento
- Que `forall` no sea un constructor de expresión, sino que introduzca nombres con tipos y ya
    + Todo constructor de expresión debería tener un `Scope` para agregar, en caso de que era un `forall` eliminado y agregando nombres
    + (capaz lo mismo para `exists`)
- Para funciones mutualmente recursivas: [idris tutorial](http://docs.idris-lang.org/en/latest/tutorial/typesfuns.html#note-declaration-order-and-mutual-blocks)
- Las instancias de comportamientos pueden generar funciones `__<type>__<behaviour>__<scope>`, ya que dos `_` seguidos están prohíbidos para identificadores.

## A discutir

- ¿Deberíamos realmente tener alcances cerrados y abiertos?
