# Práctica: Préstamo de equipo en Odoo

- **Nombre:** José Vidal Cisneros Maradiaga
- **Número de cuenta:** 0801200316240
- **Asignatura:** Programación de Negocios · Odoo 20.0
- **Módulo:** `prestamo_unicah`

## Respuestas

### Paso 1. Catálogo de equipos

1. **¿Qué pasa con un equipo cuando se desmarca el campo Activo? ¿Por qué es mejor que borrarlo?**
   Al desmarcar `active` el equipo queda *archivado*: deja de aparecer en las listas, el kanban y los selectores
   (Odoo filtra por `active = True` automáticamente), pero el registro sigue en la base de datos y se puede
   recuperar con el filtro "Archivado". Es mejor que borrarlo porque se conserva el historial: los préstamos
   anteriores que usaron ese equipo siguen teniendo su referencia, y no se pierde información ni se rompen
   relaciones con otros registros.

2. **¿Para qué sirve `copy=False` en el campo `codigo`?**
   Cuando se duplica un equipo (Acción > Duplicar), los campos con `copy=False` no se copian. Así el duplicado
   sale sin código de inventario y obliga a escribir uno nuevo, porque el código identifica a un equipo físico
   y no debe repetirse.

### Paso 2. Registro de préstamos

1. **Este modelo usa `_name` y `_inherit` al mismo tiempo. ¿Qué hace cada uno aquí?**
   `_name = "prestamo.prestamo"` define un modelo **nuevo** (con su propia tabla). `_inherit = ["mail.thread"]`,
   al usarse junto con un `_name` distinto, hace que el modelo nuevo *incorpore* los campos y métodos de la
   clase `mail.thread` (chatter, `message_post`, seguimiento de cambios). Es herencia por "mixin": no modifica
   `mail.thread`, solo copia su funcionalidad al modelo nuevo.

2. **¿Qué hace `tracking=True` en el campo `state`?**
   Hace que cada vez que cambia el valor de `state`, Odoo escriba automáticamente en el chatter un mensaje con
   el valor anterior, el nuevo, el usuario y la fecha. Solo funciona porque el modelo hereda de `mail.thread`.

### Paso 3. Equipos del préstamo

1. **En `fields.One2many("prestamo.linea", "prestamo_id")`, ¿qué indica cada parámetro?**
   El primero (`"prestamo.linea"`) es el modelo hijo, es decir, el tipo de registros que contiene la lista.
   El segundo (`"prestamo_id"`) es el nombre del campo `Many2one` dentro de `prestamo.linea` que apunta de
   regreso al préstamo; Odoo lo usa para saber qué líneas pertenecen a cada préstamo. El `One2many` no guarda
   datos propios: es la vista inversa de ese `Many2one`.

2. **¿Qué pasa con las líneas cuando se borra un préstamo? ¿Qué parte del código lo define?**
   Se borran junto con el préstamo (borrado en cascada). Lo define `ondelete="cascade"` en el campo
   `prestamo_id` de `prestamo.linea`.

### Paso 4. Flujo de estados

1. **Los botones ya se ocultan según el estado. ¿Por qué los métodos vuelven a validar el estado?**
   `invisible` solo afecta a la interfaz. El método se puede ejecutar por otras vías: una pantalla
   desactualizada, un doble clic, la API/RPC, otro código o una acción automática. La validación en el servidor
   garantiza que las reglas se cumplan siempre, no solo cuando el botón está visible.

2. **¿Qué hace `prestamo.linea_ids.equipo_id.write(...)`? ¿Por qué no hace falta un `for` para cada equipo?**
   `linea_ids` es el conjunto de líneas del préstamo; al pedir `.equipo_id` sobre ese conjunto, Odoo devuelve
   el conjunto de todos los equipos de esas líneas. Luego `.write({"estado": "prestado"})` actualiza el campo
   en todos ellos de una sola vez. Los métodos de Odoo trabajan sobre *recordsets* (varios registros), por eso
   no se necesita recorrer uno por uno.

### Paso 5. Reglas del préstamo

1. **El campo `equipo_id` ya filtra los equipos disponibles. ¿Por qué hace falta el TODO 5?**
   El `domain` solo filtra la lista desplegable en el momento en que se elige el equipo. No impide que, después,
   el equipo deje de estar disponible (por ejemplo, se envía a mantenimiento o lo toma otro préstamo) antes de
   pulsar Prestar. Tampoco se aplica si el registro se crea por importación o por código. El TODO 5 revisa el
   estado real del equipo justo en el momento de prestar.

2. **¿En qué momento se ejecuta un método con `@api.constrains`? ¿Y la validación de `action_prestar`?**
   Un `@api.constrains` se ejecuta automáticamente al crear o guardar (create/write) un registro cuando cambia
   alguno de los campos indicados, sin importar si viene de la interfaz, de una importación o de código; si
   falla, se cancela el guardado. La validación de `action_prestar` solo corre cuando alguien pulsa el botón
   Prestar (o llama al método).
