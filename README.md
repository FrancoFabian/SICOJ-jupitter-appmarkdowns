# Requerimientos Funcionales - Filtros

Este documento describe los requerimientos funcionales **RN12** y **RN13** relacionados con los filtros aplicables en el sistema.

---

## RN12 - Filtro: Estado de la tarea

- **Tipo:** \"Lista con opciones\"
- **Habilitado:** \"Sí\"
- **Obligatorio:** \"No aplica\"
- **Validaciones:**
  - El sistema debe permitir elegir cualquier estado de la tarea **excepto** los que se muestran en la **bandeja de pendientes**.

> [!NOTE] 
> La restricción mencionada implica que los estados presentes en la bandeja de pendientes no estarán disponibles para selección en este filtro.

> [!WARNING]  
> **Advertencia técnica:**  
> Este endpoint ha sido probado únicamente en **mi entorno de trabajo**, bajo el rol (Administrador AUA / Global / Abogado, según el contexto)**.  
> 
> Se ha detectado que **no funciona correctamente cuando lo usa frontend A en otro entorno de trabajo** cuando se usa con el **ID del Administrador**.  
> 
> También se **desconoce si el filtro `ByIdRol` realmente está implementado en el backend**, aunque **en mi caso sí funciona correctamente**.  
> 




### Ejemplo de Solicitud a la API

Para obtener la lista de estados de tarea aplicables para un rol específico (por ejemplo, con `idRol = 3`), se realiza la siguiente solicitud:

```http
GET http://127.0.0.1:8000/sicoj/catalogos/api/v1/estado-tarea-ap/get-list/?&filters[0][ByIdRol]=3

```

#### Respuesta
```json
{
    "result": {
        "3": "ASIGNADO",
        "8": "CONCLUIDO",
        "6": "CONCLUIDO POR IMPROCEDENCIA",
        "5": "CONCLUIDO REMITIDO",
        "1": "PENDIENTE DE TURNAR",
        "7": "REASIGNADO",
        "4": "REMITIDO"
    },
    "success": true,
    "messages": []
}
```
> **Nota:** En este ejemplo, el estado \"1\": \"PENDIENTE DE TURNAR\" corresponde a un estado presente en la bandeja de pendientes y, por lo tanto, debe ser excluido del filtro en el frontend, y por otro lado falta "PENDIENTE DE TURNAR".
## RN13 - Filtro: Estado procesal

- **Tipo:** \"Lista de opciones\"
- **Habilitado:** \"Sí\"
- **Obligatorio:** \"No aplica\"
- **Validaciones:**
  - El sistema debe permitir elegir cualquier estado procesal **excepto** los que se muestran en la **bandeja de pendientes**.

> **Nota:** Similar al filtro anterior, los estados procesales que aparecen en la bandeja de pendientes deben ser excluidos de este filtro.

### Ejemplo de Solicitud a la API

Para obtener la lista de estados procesales aplicables para un rol específico (por ejemplo, con `idRol = 3`), se realiza la siguiente solicitud:

```http
GET http://127.0.0.1:8000/sicoj/catalogos/api/v1/estado-procesal-ap/get-list/?&filters[0][ByIdRol]=3
```
#### Respuesta

```json
{
    "result": {
        "1": "ACTIVO",
        "8": "CONCLUIDO_POR_FACULTAD_DE_ABSTENERSE_DE_INVESTIGAR",
        "6": "CONCLUIDO POR IMPROCEDENCIA",
        "4": "CONCLUIDO POR REMISIÓN",
        "7": "INVESTIGACIÓN INICIAL",
        "5": "PROCEDENTE",
        "3": "REMITIDO"
    },
    "success": true,
    "messages": []
}
```
> ***Nota:*** En este caso, se debe identificar cuáles de estos estados están presentes en la bandeja de pendientes para excluirlos del filtro en el frontend.


### Implementación en el Frontend
En el frontend, se utilizan funciones para gestionar las solicitudes a la API y procesar las respuestas, aplicando los filtros necesarios según los requerimientos.

#### Gestión de Estados Procesales
```ts

manageEstadoProcesal(CRUD_ACTION.LIST, {
    filters: {
        idRol: TIPO_ROL.OP
    }
})
.then((result) => {
    listEstadoProcesal = util.objectToListValue(result || {}) || [{}];
})
.catch((ex) => {
    showAppDialogConfirm(
        \"Mensaje\",
        `No fue posible obtener el catálogo de estados procesales. ${ex || \"\"}`
    );
});
```
> ***Nota Técnica*** En esta función, se solicita a la API el catálogo de estados procesales filtrados por el rol operativo (TIPO_ROL.OP). La respuesta se convierte en una lista de valores y se almacena en listEstadoProcesal. En caso de error, se muestra un mensaje al usuario.


#### Gestión de Estados de Tarea
```ts
manageTarea(CRUD_ACTION.LIST, {
    filters: {
        idRol: TIPO_ROL.OP
    }
})
.then((result) => {
    const listTask = util.objectToListValue(result || {}) || [{}];
    listManageTarea = listTask.filter((task) => task.value !== \"1\");
    console.log(listTask);
})
.catch((ex) => {
    showAppDialogConfirm(
        \"Mensaje\",
        `No fue posible obtener el catálogo de estados de tarea. ${ex || \"\"}`
    );
});
```
> ***Nota Técnica:*** Similar a la función anterior, se solicita a la API el catálogo de estados de tarea filtrados por el rol operativo. La respuesta se convierte en una lista de tareas (listTask). Posteriormente, se filtran las tareas para excluir aquellas con value igual a \"1\" (correspondiente a "PENDIENTE DE TURNAR") lo cual me prohibieron implementar xD, almacenando el resultado en listManageTarea. En caso de error, se muestra un mensaje al usuario.


### Historias de Usuario para Historico de asuntos Filtros Estado procesal y Tarea


| Sección      | Requerimiento | Filtro               | Tipo                  | Habilitado | Obligatorio | Validaciones                                                                 |
|--------------|---------------|----------------------|-----------------------|------------|-------------|------------------------------------------------------------------------------|
| **Administrador** | RN12          | Estado de la tarea     | Lista con opciones     | Sí         | No aplica    | El sistema debe permitir elegir cualquier estado de la tarea excepto las que se muestran en la bandeja de pendientes. |
| **Administrador** | RN13          | Estado procesal        | Lista de opciones      | Sí         | No aplica    | El sistema debe permitir elegir cualquier estado procesal excepto las que se muestran en la bandeja de pendientes.     |
| **Abogado**       | RN12          | Estado de la tarea     | Lista con opciones     | Sí         | No aplica    | El sistema debe presentar la siguiente opción: Concluido.                           |
| **Abogado**       | RN13          | Estado procesal        | Lista de opciones      | Sí         | No aplica    | El sistema debe mostrar y permitir elegir cualquier estado procesal asociado a la conclusión del asunto.               |
| **Administrador Global** | RN13 | Estado de la tarea | Lista con opciones | Sí | No aplica | El sistema debe permitir elegir cualquier estado de la tarea excepto las que se muestran en la bandeja de pendientes. |
| **Administrador Global** | RN14 | Estado procesal | Lista de opciones | Sí | No aplica | El sistema debe permitir elegir cualquier estado procesal excepto las que se muestran en la bandeja de pendientes. |
| **Administrador AUA** | RN12 | Estado de la tarea | Lista con opciones | Sí | No aplica | El sistema debe permitir elegir cualquier estado de la tarea excepto las que se muestran en la bandeja de pendientes. |
| **Administrador AUA** | RN13 | Estado procesal    | Lista de opciones  | Sí | No aplica | El sistema debe permitir elegir cualquier estado procesal excepto las que se muestran en la bandeja de pendientes.     |
| **OP** | RN12 | Estado de la tarea | Lista con opciones | Sí | No aplica | El sistema debe permitir elegir cualquier estado de la tarea excepto las que se muestran en la bandeja de pendientes. |
| **OP** | RN13 | Estado procesal    | Lista de opciones  | Sí | No aplica | El sistema debe permitir elegir cualquier estado procesal excepto las que se muestran en la bandeja de pendientes.     |

#### sicoj.tc_estado_tarea_ap

| id | nombre                    | descripcion            | fecha_creacion                | fecha_modificacion             | fecha_inicio | activo |
|----|----------------------------|-------------------------|-------------------------------|-------------------------------|--------------|--------|
| 1  | PENDIENTE DE TURNAR        |                         | 2025-03-19 01:55:41.574349+00 | 2025-03-19 01:55:41.574348+00 | 2025-03-18   | true   |
| 2  | PENDIENTE DE ASIGNAR       |                         | 2025-03-19 01:55:41.574361+00 | 2025-03-19 01:55:41.574361+00 | 2025-03-18   | true   |
| 3  | ASIGNADO                   |                         | 2025-03-19 01:55:41.574366+00 | 2025-03-19 01:55:41.574366+00 | 2025-03-18   | true   |
| 4  | REMITIDO                   |                         | 2025-03-19 01:55:41.574369+00 | 2025-03-19 01:55:41.574369+00 | 2025-03-18   | true   |
| 5  | CONCLUIDO REMITIDO         |                         | 2025-03-19 01:55:41.574373+00 | 2025-03-19 01:55:41.574372+00 | 2025-03-18   | true   |
| 6  | CONCLUIDO POR IMPROCEDENCIA|                         | 2025-03-19 01:55:41.574381+00 | 2025-03-19 01:55:41.574381+00 | 2025-03-18   | true   |
| 7  | REASIGNADO                 |                         | 2025-03-19 01:55:41.574626+00 | 2025-03-19 01:55:41.574625+00 | 2025-03-18   | true   |
| 8  | CONCLUIDO                  |                         | 2025-03-20 23:35:00.121847+00 | 2025-03-20 23:35:00.121847+00 | 2025-03-20   | true   |


### sicoj.tc_estado_procesal_ap

| id | nombre                                                      | descripcion | fecha_creacion                | fecha_modificacion             | fecha_inicio | activo |
|----|--------------------------------------------------------------|-------------|-------------------------------|-------------------------------|--------------|--------|
| 1  | ACTIVO                                                      |             | 2025-03-19 01:55:41.593758+00 | 2025-03-19 01:55:41.593758+00 | 2025-03-18   | true   |
| 2  | EN ESTUDIO                                                  |             | 2025-03-19 01:55:41.593777+00 | 2025-03-19 01:55:41.593777+00 | 2025-03-18   | true   |
| 3  | REMITIDO                                                    |             | 2025-03-19 01:55:41.593774+00 | 2025-03-19 01:55:41.593774+00 | 2025-03-18   | true   |
| 4  | CONCLUIDO POR REMISIÓN                                      |             | 2025-03-19 01:55:41.593777+00 | 2025-03-19 01:55:41.593777+00 | 2025-03-18   | true   |
| 5  | PROCEDENTE                                                  |             | 2025-03-19 01:55:41.593781+00 | 2025-03-19 01:55:41.593781+00 | 2025-03-18   | true   |
| 6  | CONCLUIDO POR IMPROCEDENCIA                                 |             | 2025-03-19 01:55:41.593788+00 | 2025-03-19 01:55:41.593788+00 | 2025-03-18   | true   |
| 7  | INVESTIGACIÓN INICIAL                                       |             | 2025-03-19 01:55:41.593788+00 | 2025-03-19 01:55:41.593788+00 | 2025-03-18   | true   |
| 8  | CONCLUIDO_POR_FACULTAD_DE_ABSTENERSE_DE_INVESTIGAR          |             | 2025-03-20 23:21:10.593825+00 | 2025-03-20 23:21:10.593825+00 | 2025-03-20   | true   |
| 9  | CONCLUIDO_POR_ARCHIVO_TEMPORAL                              |             | 2025-03-20 23:21:10.593825+00 | 2025-03-20 23:21:10.593825+00 | 2025-03-20   | true   |
| 10 | CONCLUIDO_POR_NO_EJERCICIO_DE_LA_ACCION_PENAL               |             | 2025-03-20 23:21:10.593825+00 | 2025-03-20 23:21:10.593825+00 | 2025-03-20   | true   |
| 11 | CONCLUIDO_POR_ACUERDO_REPARATORIO                           |             | 2025-03-20 23:21:10.593825+00 | 2025-03-20 23:21:10.593825+00 | 2025-03-20   | true   |
| 12 | CONCLUIDO_POR_CRITERIO_DE_OPORTUNIDAD                       |             | 2025-03-20 23:21:10.593825+00 | 2025-03-20 23:21:10.593825+00 | 2025-03-20   | true   |
| 13 | INVESTIGACION_COMPLEMENTARIA                                |             | 2025-03-20 23:21:10.593825+00 | 2025-03-20 23:21:10.593825+00 | 2025-03-20   | true   |
| 14 | ETAPA_INTERMEDIA                                            |             | 2025-03-20 23:21:10.593825+00 | 2025-03-20 23:21:10.593825+00 | 2025-03-20   | true   |
| 15 | CONCLUIDO_POR_SENTENCIA_CONDENATORIA_EN_PROCEDIMIENTO_ABREVIADO |          | 2025-03-20 23:21:10.593825+00 | 2025-03-20 23:21:10.593825+00 | 2025-03-20   | true   |
| 16 | CONCLUIDO_POR_SOBRESEIMIENTO                                |             | 2025-03-20 23:21:10.593825+00 | 2025-03-20 23:21:10.593825+00 | 2025-03-20   | true   |
| 17 | CONCLUIDO_POR_SUSPENSIÓN_CONDICIONAL_DEL_PROCESO            |             | 2025-03-20 23:21:10.593825+00 | 2025-03-20 23:21:10.593825+00 | 2025-03-20   | true   |
| 18 | ETAPA_JUICIOS                                               |             | 2025-03-20 23:21:10.593825+00 | 2025-03-20 23:21:10.593825+00 | 2025-03-20   | true   |
| 19 | CONCLUIDO_POR_SENTENCIA_CONDENATORIA                        |             | 2025-03-20 23:21:10.593825+00 | 2025-03-20 23:21:10.593825+00 | 2025-03-20   | true   |
| 20 | CONCLUIDO_POR_SENTENCIA_ABSOLUTORIA                         |             | 2025-03-20 23:21:10.593825+00 | 2025-03-20 23:21:10.593825+00 | 2025-03-20   | true   |
| 21 | SENTENCIA       |             | 2025-03-20 23:21:10.593825+00 | 2025-03-20 23:21:10.593825+00 | 2025-03-20 | true |
| 22 | RESUELTO        |             | 2025-03-20 23:21:10.593825+00 | 2025-03-20 23:21:10.593825+00 | 2025-03-20 | true |
| 23 | EN_REPARACION   |             | 2025-03-20 23:21:10.593825+00 | 2025-03-20 23:21:10.593825+00 | 2025-03-20 | true |
> [!IMPORTANT]
> **Consulta técnica sobre filtrado por `ByIdRol`**
>
> Actualmente, se está utilizando el filtro `ByIdRol` en las solicitudes hacia el backend, pero no se ha podido confirmar si este filtro **realmente existe o está validado en la lógica del servidor**.
>
>  Me gustaría saber:
> - ¿Existe formalmente el soporte para `ByIdRol` en el backend?
> - En caso de que no exista, ¿hay alguna otra manera oficial o recomendada de obtener listas filtradas por rol directamente desde el backend?
> - Si no hay soporte aún, ¿sería posible implementarlo? Y de ser así, ¿qué requisitos se deben cumplir o a quién debo dirigirme?
>
> 💬 Alejandra me comentó amablemente que **no debería realizar el filtrado manual desde el frontend para todas las listas**, ya que eso puede impactar negativamente en el rendimiento de los equipos de los usuarios.
>
> Por lo tanto, solo busco confirmar si este comportamiento está soportado o si sería mejor tratarlo desde el backend para que la experiencia del usuario final sea más fluida y eficiente.







