# Prueba técnica Fullstack Junior — Pymedesk

Aplicación pequeña de gestión de pedidos con Flask, SQLite y JavaScript nativo. Tiempo estimado: **30–45 minutos**. Puedes usar asistentes de IA.

## Preparación

Requiere Python 3.11 o superior. En la raíz del repositorio:

```bash
python -m venv .venv
```

Activa el entorno virtual con `.venv\Scripts\activate` en Windows o `source .venv/bin/activate` en macOS/Linux. Después:

```bash
pip install -r requirements.txt
python seed.py
python app.py
```

Abre <http://127.0.0.1:5000>. El seed se puede ejecutar más de una vez sin duplicar pedidos. La base local se crea en `data/orders.sqlite3` y no se incluye en Git.

## Tu tarea

1. Un pedido cancelado no debe poder pasar a enviado: el servidor debe rechazar la operación y conservar su estado cancelado. Un pedido pendiente sí debe poder pasar a enviado.
2. Cuando falle una actualización, muestra en la interfaz un mensaje visible y comprensible. Actualmente el fallo no se comunica claramente.

Si encuentras algún otro error imprevisto en la aplicación, puedes corregirlo sin necesidad de preguntar. Si no estás seguro de cómo proceder, documenta el problema y tu propuesta de solución en el repositorio.

Mantén la solución pequeña. No se requiere despliegue, documentación extensa ni pruebas automatizadas adicionales.

## API inicial

- `GET /api/orders`: devuelve todos los pedidos.
- `POST /api/orders/<id>/ship`: intenta marcar un pedido como enviado.

## Entrega

Comparte el enlace a tu repositorio con los cambios y un video de **2–4 minutos** donde muestres la aplicación, expliques el problema encontrado, tu solución y cómo comprobaste el resultado. Puedes usar asistentes de IA durante todo el ejercicio.

## Notas de la solución

### Cambios realizados

- `app.py`, endpoint `POST /api/orders/<id>/ship`: se rechaza con `409 Conflict` cualquier intento de enviar un pedido cancelado. El pedido conserva su estado y la respuesta incluye el motivo en el campo `error`.
- `templates/index.html`: se añadió un contenedor de alerta (`role="alert"`) y la función `ship` ahora lee el mensaje de error que devuelve el servidor y lo muestra. Si la respuesta no es JSON se usa un mensaje genérico, y si falla la conexión se muestra un error de red. El botón se vuelve a habilitar tras un fallo.
- `static/style.css`: estilos de la alerta.

### Decisiones

- Se usa `409 Conflict` porque la petición choca con el estado actual del recurso; no es un error de formato ni de autenticación.
- El botón "Marcar como enviado" sigue habilitado en pedidos cancelados a propósito: la regla la aplica el servidor y la interfaz comunica el motivo del rechazo, que es el objetivo de la segunda tarea.

### Mejora propuesta (no aplicada)

La comprobación del estado y el `UPDATE` se hacen en dos sentencias separadas, por lo que no son atómicos. Para que la regla la garantice la base de datos y no solo el flujo del código, se propone condicionar la actualización al estado pendiente y comprobar las filas afectadas:

```python
cur = con.execute(
    "UPDATE orders SET status = 'enviado' WHERE id = ? AND status = 'pendiente'",
    (order_id,),
)
if cur.rowcount == 0:
    # releer el pedido y responder 404 o 409 según corresponda
```

Se dejó fuera para mantener la solución pequeña, como pide el enunciado.
