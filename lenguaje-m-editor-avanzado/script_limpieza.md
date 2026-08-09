# Script de limpieza — Lenguaje M (TechStore)

Código M utilizado en el Editor Avanzado de Power Query para limpiar y tipar la tabla de ventas de TechStore.

```
let
    Origen = Table.FromRows(Json.Document(Binary.Decompress(Binary.FromText("XZBBasMwEEWvMmgdB0m2WndpJ4GWNBAaly5MFoqihcCWjGxBr5Mz9Ai+WEcOhai7mQeP/2faljCyIvAuh8kNcPQOmAAkG9cPYZLKzD8WV8YpXVOKE6e8yCjLqCDnVUs4ooMLo4Y3K7v51l+8UQ6hVEqPzhs3Rqn8J5eLnMfoRqtOXh0ctJpv9i4fPz53dYXDi0hFxhexWFKtmZyHYh/7qrRvIZK6PKP5IoqYWAXsGDrp9Qh1g6QKV+PuV6YWo4v1hOh02sLue9Le4ouaGh5bsjzx8r/nPCP60hcle3jdxpzHn5QideJp538=", BinaryEncoding.Base64), Compression.Deflate)), let t = ((type nullable text) meta [Serialized.Text = true]) in type table [idventa = t, nombreproducto = t, categoria = t, precio = t, fechaventa = _t]),  // No modificar este paso

    // Paso 2: Eliminar espacios en blanco al inicio y al final
    // de la columna nombre_producto usando Text.Trim. Sin esto,
    // " Laptop Pro 15 " y "Laptop Pro 15" quedarían como valores
    // distintos para cualquier análisis o combinación posterior.
    LimpiarEspacios = Table.TransformColumns(Origen, {{"nombre_producto", Text.Trim, type text}}),

    // Paso 3: Estandarizar la columna categoria a Title Case con
    // Text.Proper, para unificar "computación", "COMPUTACIÓN" y
    // "Computación" (y también "PRUEBA"/"prueba") bajo un único
    // formato consistente: "Computación" / "Prueba".
    EstandarizarCategoria = Table.TransformColumns(LimpiarEspacios, {{"categoria", Text.Proper, type text}}),

    // Paso 4: Filtrar y eliminar registros de prueba. Se excluyen
    // las filas donde categoria sea exactamente "Prueba" (ya en
    // Title Case gracias al paso anterior). Este paso va DESPUÉS de
    // EstandarizarCategoria a propósito: ver justificación en el
    // README (pregunta 4).
    EliminarPruebas = Table.SelectRows(EstandarizarCategoria, each [categoria] <> "Prueba"),

    // Paso 5: Definir los tipos de datos correctos para cada columna.
    // id_venta: Int64.Type | nombre_producto y categoria: type text
    // precio: type number | fecha_venta: type date
    TiparColumnas = Table.TransformColumnTypes(EliminarPruebas, {
        {"id_venta", Int64.Type},
        {"nombre_producto", type text},
        {"categoria", type text},
        {"precio", type number},
        {"fecha_venta", type date}
    }, "en-US")
in
    TiparColumnas
```
