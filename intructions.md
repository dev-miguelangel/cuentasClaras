# CuentasRapidas
Sistema para organizar cuentas utilizando modelo de desarrollo agil apollado con un tablero kanban que permita registrar priorizar y organizar gastos.

## Flujo de trabajo
- ingresar con google
- registrar gasto
- mover cuentas en un tablero kanban que tendrá las columnas, "Por pagar", "Carrito", "pagado"

## Reglas
- los gastos tendrán, titulo, monto (en pesos chilenos, no acepta decimales) y estado.
- los estados posibles deben ser coherentes con las columnas del kanban
- las columnas del kanban serán "Por pagar", "Carrito", "pagado" (orden estricto de izquierda a derecha)
- abajo del título de cada columna debe mostrar el total en pesos chilenos
- cada gasto será una card y se debe poder arrastrar y soltar entre las columnas

## UI
- las vistas deben estar inspiradas en trello utilizando las mejores practicas de ux