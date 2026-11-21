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
