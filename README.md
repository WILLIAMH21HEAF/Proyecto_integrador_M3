# Proyecto Integrador M3 - Opción C: Agente de Soporte

## 1. Nombre del proyecto y opción elegida

**Nombre del proyecto:** Agente de Soporte con LangGraph y Human-in-the-loop

**Opción elegida:** Opción C - Agente de soporte

Este proyecto corresponde al Proyecto Integrador M3 de la especialización de IA Generativa. La solución implementa un agente de soporte basado en LangGraph, cuyo objetivo es gestionar consultas de usuarios mediante un flujo con estado, checkpoints, trazabilidad y aprobación humana cuando el caso es sensible.

---

## 2. Descripción breve del caso de negocio

En un proceso de atención al cliente, los usuarios pueden registrar consultas relacionadas con problemas técnicos, facturación, acceso a cuenta u otros temas generales. Algunas consultas pueden resolverse automáticamente, pero otras requieren revisión humana antes de enviar una respuesta final, especialmente cuando involucran reclamos, cobros duplicados, devoluciones, denuncias o temas sensibles.

El proyecto modela este proceso como un grafo de LangGraph. El agente recibe una consulta, clasifica la intención, busca información en una base de conocimiento simulada, propone una respuesta y evalúa si el caso requiere aprobación humana. Si el caso es sensible, el flujo se pausa, guarda un checkpoint y continúa cuando se recibe una decisión humana simulada.

---

## 3. Diagrama o descripción del flujo del grafo

El flujo principal del grafo es el siguiente:

```text
START
  ↓
recibir_consulta
  ↓
clasificar_intencion
  ↓
buscar_informacion
  ↓
proponer_respuesta
  ↓
¿requiere_aprobacion?
  ├── No → enviar_respuesta_final → END
  └── Sí → aprobacion_humana → enviar_respuesta_final → END
```

### Descripción del flujo

1. **Recibir consulta:** Se registra el ticket inicial del usuario.
2. **Clasificar intención:** Se identifica el tipo de consulta: soporte técnico, facturación, cuenta o general.
3. **Buscar información:** Se recupera información desde una base de conocimiento simulada.
4. **Proponer respuesta:** Se genera una respuesta preliminar.
5. **Evaluar aprobación humana:** Si el caso es sensible, el flujo se pausa.
6. **Aprobación humana:** Se simula la revisión de una persona, que puede aprobar, rechazar o corregir la respuesta.
7. **Enviar respuesta final:** Se genera la respuesta final según el resultado del flujo.

---

## 4. Explicación del estado y campos principales

El estado del grafo se define mediante `TypedDict` en la clase `SupportState`.

```python
class SupportState(TypedDict):
    ticket_id: str
    usuario: str
    consulta: str

    intencion: NotRequired[str]
    contexto_recuperado: NotRequired[str]
    respuesta_propuesta: NotRequired[str]
    requiere_aprobacion: NotRequired[bool]
    decision_humana: NotRequired[dict[str, Any]]
    respuesta_final: NotRequired[str]
    estado_ticket: NotRequired[str]

    eventos: Annotated[list[str], add]
```

### Campos principales

| Campo | Descripción |
|---|---|
| `ticket_id` | Identificador único del ticket de soporte. |
| `usuario` | Nombre del usuario que registra la consulta. |
| `consulta` | Texto original ingresado por el usuario. |
| `intencion` | Categoría detectada de la consulta. |
| `contexto_recuperado` | Información recuperada desde la base de conocimiento simulada. |
| `respuesta_propuesta` | Respuesta preliminar generada por el agente. |
| `requiere_aprobacion` | Indicador booleano que define si el caso debe ser revisado por una persona. |
| `decision_humana` | Resultado de la revisión humana simulada. |
| `respuesta_final` | Respuesta final que se enviaría al usuario. |
| `estado_ticket` | Estado actual del ticket dentro del flujo. |
| `eventos` | Lista de eventos que registra la trazabilidad de la ejecución. |

El campo `eventos` usa `Annotated[list[str], add]`, lo que permite acumular mensajes de trazabilidad durante la ejecución del grafo.

---

## 5. Lista de nodos y responsabilidad de cada uno

| Nodo | Responsabilidad |
|---|---|
| `recibir_consulta` | Registra la recepción del ticket y actualiza el estado inicial. |
| `clasificar_intencion` | Clasifica la consulta del usuario en una intención: soporte técnico, facturación, cuenta o general. |
| `buscar_informacion` | Recupera información desde una base de conocimiento simulada según la intención detectada. |
| `proponer_respuesta` | Genera una respuesta preliminar y determina si el caso requiere aprobación humana. |
| `aprobacion_humana` | Pausa el flujo mediante `interrupt` y espera una decisión humana simulada. |
| `enviar_respuesta_final` | Genera la respuesta final, considerando si hubo aprobación, corrección o rechazo humano. |

---

## 6. Explicación de checkpoints y `thread_id`

El proyecto utiliza `InMemorySaver` como checkpointer:

```python
checkpointer = InMemorySaver()
graph = builder.compile(checkpointer=checkpointer)
```

El checkpointer permite que LangGraph conserve el estado del flujo durante la ejecución. Esto es necesario para demostrar la pausa y continuación desde un punto guardado.

Cada ejecución del grafo se asocia a un `thread_id` identificable:

```python
config_sensible = {
    "configurable": {
        "thread_id": "thread-ticket-sensible-002"
    }
}
```

El `thread_id` permite recuperar el estado, revisar el historial de checkpoints y continuar una ejecución pausada. En el caso sensible, el flujo se pausa en el nodo `aprobacion_humana` y luego continúa desde ese mismo checkpoint usando:

```python
Command(resume=decision_aprobador)
```

---

## 7. Instrucciones de instalación

### Opción 1: instalación desde notebook

Ejecutar la primera celda del notebook:

```python
%%capture
!pip install -q langgraph typing_extensions
```

### Opción 2: instalación desde terminal

Instalar las dependencias con:

```bash
pip install -r requirements.txt
```

Contenido recomendado de `requirements.txt`:

```txt
langgraph
typing_extensions
ipython
```

---

## 8. Instrucciones de ejecución

### Ejecución en notebook

1. Abrir el archivo:

```text
ProyectoM3.ipynb
```

2. Ejecutar las celdas en orden.
3. Revisar primero la construcción del grafo.
4. Ejecutar la demo normal.
5. Ejecutar la demo sensible con aprobación humana.
6. Revisar el estado final y el historial de checkpoints.

### Ejecución esperada

El notebook incluye tres escenarios:

| Demo | Descripción |
|---|---|
| Demo 1 | Caso normal sin aprobación humana. |
| Demo 2 | Caso sensible con pausa, aprobación humana y continuación desde checkpoint. |
| Demo 3 | Caso sensible donde la respuesta no es aprobada y el ticket se deriva a revisión. |

---

## 9. Cómo reproducir la aprobación humana

La Opción C requiere demostrar Human-in-the-loop. En este proyecto se reproduce de la siguiente manera:

### Paso 1: ejecutar un caso sensible

```python
ticket_sensible = {
    "ticket_id": "TK-002",
    "usuario": "María López",
    "consulta": (
        "Me hicieron un cobro duplicado y exijo devolución inmediata. "
        "Esto es un reclamo urgente."
    ),
    "eventos": []
}

config_sensible = {
    "configurable": {
        "thread_id": "thread-ticket-sensible-002"
    }
}

resultado_pausa = support_graph.invoke(ticket_sensible, config_sensible)
```

El flujo se detiene en el nodo `aprobacion_humana` porque el caso contiene términos sensibles como `cobro duplicado`, `devolución` y `reclamo`.

### Paso 2: revisar el estado pausado

```python
estado_pausado = support_graph.get_state(config_sensible)

pprint(estado_pausado.values)
print(estado_pausado.next)
```

Esto permite verificar que el grafo quedó pausado y que existe un checkpoint asociado al `thread_id`.

### Paso 3: simular la aprobación humana

```python
decision_aprobador = {
    "aprobado": True,
    "comentario": (
        "Se aprueba la respuesta, pero debe evitar prometer devolución inmediata."
    ),
    "respuesta_corregida": (
        "Hemos registrado tu reclamo por posible cobro duplicado. "
        "El caso será revisado por el equipo de facturación validando comprobante, "
        "monto y fecha del cargo. Luego de la revisión, se te informará el resultado "
        "por el canal registrado."
    )
}
```

### Paso 4: continuar desde el checkpoint

```python
resultado_final_sensible = support_graph.invoke(
    Command(resume=decision_aprobador),
    config_sensible
)
```

Con `Command(resume=...)`, el grafo continúa desde el punto pausado y genera la respuesta final.

---

## 10. Aclaración sobre Human-in-the-loop

Este proyecto desarrolla la **Opción C: Agente de soporte**. Por lo tanto, la capacidad clave implementada es:

```text
Checkpoint y continuación luego de aprobación humana.
```

En este proyecto se implementa Human-in-the-loop mediante:

```python
interrupt(...)
```

y la continuación desde checkpoint mediante:

```python
Command(resume=...)
```

Esto permite pausar el flujo, recibir una aprobación o corrección humana simulada y continuar desde el punto guardado.

---

## 11. Capturas o resultado esperado de la demo

### Resultado esperado - Demo 1

Para un caso normal, el estado final debe mostrar:

```text
estado_ticket = "respondido_automaticamente"
requiere_aprobacion = False
respuesta_final = respuesta propuesta por el agente
```

Eventos esperados:

```text
Ticket recibido
Intención clasificada
Contexto recuperado
Respuesta propuesta generada
Respuesta final enviada automáticamente
```

### Resultado esperado - Demo 2

Para un caso sensible aprobado, el flujo debe mostrar:

```text
estado_ticket = "respondido_con_aprobacion"
requiere_aprobacion = True
decision_humana.aprobado = True
respuesta_final = respuesta corregida o aprobada por el humano
```

Eventos esperados:

```text
Ticket recibido
Intención clasificada
Contexto recuperado
Respuesta propuesta generada
Caso pausado para aprobación humana
Decisión humana recibida
Respuesta final enviada luego de aprobación humana
```

### Resultado esperado - Demo 3

Para un caso sensible no aprobado, el estado final debe mostrar:

```text
estado_ticket = "derivado_a_revision"
requiere_aprobacion = True
decision_humana.aprobado = False
respuesta_final = mensaje de derivación a especialista
```

Eventos esperados:

```text
Ticket recibido
Intención clasificada
Contexto recuperado
Respuesta propuesta generada
Decisión humana recibida
La respuesta no fue aprobada
Ticket derivado a revisión
```

---




