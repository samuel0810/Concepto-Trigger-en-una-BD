# Documentación sobre Triggers en Bases de Datos

Este documento explora en detalle el concepto de Triggers en bases de datos, su funcionalidad, estructura, tipos y cuándo se ejecutan.

---

## 1. ¿Qué es un Trigger?

Un **Trigger** (también conocido como Disparador) es un tipo especial de procedimiento almacenado que se ejecuta automáticamente o "se dispara" cuando ocurre un evento específico en una base de datos. Estos eventos suelen ser operaciones de manipulación de datos (DML) como `INSERT`, `UPDATE` o `DELETE` en una tabla, pero también pueden ser eventos DDL (Data Definition Language) o eventos relacionados con la base de datos (por ejemplo, el inicio o el cierre de una sesión).

A diferencia de los procedimientos almacenados normales, que se invocan explícitamente, un trigger se asocia a una tabla y se ejecuta implícitamente cuando se cumple la condición de su activación.

## 2. ¿Para qué sirve un Trigger?

Los triggers son herramientas poderosas para mantener la integridad y la consistencia de los datos, así como para automatizar tareas. Sus principales usos son:

* **Aplicar reglas de negocio complejas:** Pueden imponer reglas que no son posibles de aplicar con restricciones de integridad declarativas (como `CHECK` o `FOREIGN KEY`). Por ejemplo, asegurar que el saldo de una cuenta nunca sea negativo después de una transacción.
* **Auditoría y registro de cambios:** Registrar quién, cuándo y qué datos se modificaron. Esto es crucial para la seguridad y la trazabilidad.
* **Mantenimiento de la integridad referencial compleja:** Manejar situaciones donde las claves foráneas simples no son suficientes, como cascadas personalizadas de actualizaciones o eliminaciones.
* **Sincronización de datos:** Mantener la consistencia entre diferentes tablas o incluso entre diferentes bases de datos.
* **Automatización de tareas:** Ejecutar acciones adicionales basadas en eventos de datos, como enviar una notificación por correo electrónico cuando el stock de un producto baja de cierto nivel.
* **Validación de datos:** Realizar validaciones de datos más sofisticadas antes de que una operación DML se complete.

## 3. ¿Cuándo se puede usar un Trigger?

Se puede usar un trigger en diversas situaciones donde se necesita una acción automatizada o una validación compleja ligada a eventos específicos en la base de datos. Algunos ejemplos son:

* **Antes de una inserción/actualización/eliminación (`BEFORE`):** Para validar datos, modificar valores antes de que se almacenen, o evitar la operación si no cumple ciertas condiciones.
    * **Caso de uso:** Asegurar que un precio de venta sea siempre mayor que el costo antes de insertar un producto.
* **Después de una inserción/actualización/eliminación (`AFTER`):** Para realizar acciones subsecuentes a la operación, como registrar la operación en una tabla de auditoría, actualizar datos en otras tablas, o disparar eventos externos.
    * **Caso de uso:** Registrar en una tabla de `historial_transacciones` cada vez que se realiza un depósito o retiro en una cuenta bancaria.
* **En eventos DDL:** Para auditar cambios en la estructura de la base de datos (creación de tablas, modificación de vistas, etc.).
    * **Caso de uso:** Registrar quién y cuándo alteró una tabla en el esquema.
* **En eventos de la base de datos:** Para realizar acciones cuando la base de datos se inicia o se detiene, o cuando un usuario inicia o cierra sesión.
    * **Caso de uso:** Restringir el número de sesiones activas por un usuario en particular.

## 4. Estructura de un Trigger

La estructura de un trigger puede variar ligeramente entre diferentes sistemas de gestión de bases de datos (SGBD) como SQL Server, MySQL, PostgreSQL u Oracle. Sin embargo, los componentes clave son los siguientes:

* **Momento de disparo:** `BEFORE` o `AFTER` (o `INSTEAD OF` en algunos SGBD para vistas).
* **Evento de disparo:** `INSERT`, `UPDATE`, `DELETE` (o `CREATE`, `ALTER`, `DROP` para triggers DDL).
* **Tabla/Vista asociada:** La tabla o vista sobre la que opera el trigger.
* **Nivel de granularidad:**
    * **`FOR EACH ROW` (o `ROW LEVEL`):** El trigger se ejecuta una vez por cada fila afectada por la operación DML. Es el más común para la mayoría de los casos de uso.
    * **`FOR EACH STATEMENT` (o `STATEMENT LEVEL`):** El trigger se ejecuta una vez por cada sentencia DML, independientemente del número de filas afectadas. Útil para auditorías a nivel de sentencia o para controlar el acceso a la tabla.
* **Cuerpo del trigger:** El bloque de código (generalmente SQL) que se ejecuta cuando el trigger se dispara. Puede acceder a los datos `OLD` (valores antes de la modificación) y `NEW` (valores después de la modificación) en triggers a nivel de fila.

### Ejemplo de estructura (conceptual - para MySQL/PostgreSQL):

```sql
CREATE TRIGGER nombre_trigger
    [BEFORE | AFTER] [INSERT | UPDATE | DELETE]
    ON nombre_tabla
    FOR EACH ROW  -- O FOR EACH STATEMENT (en algunos SGBD) 
    [WHEN (condicion)] -- Condición opcional para disparar el trigger
BEGIN
    -- Cuerpo del trigger: Sentencias SQL
    -- Se pueden usar OLD.columna y NEW.columna en triggers de fila
END;
```

## 5. Tipos de Trigger

Los triggers se pueden clasificar principalmente por el momento y el evento en que se disparan:

1.  **Triggers DML (Data Manipulation Language):** Son los más comunes y se activan por operaciones `INSERT`, `UPDATE` o `DELETE` en tablas.
    * **`BEFORE INSERT`:** Se ejecuta antes de insertar una fila. Útil para validar datos o modificar valores antes de la inserción.
    * **`AFTER INSERT`:** Se ejecuta después de insertar una fila. Útil para actualizar otras tablas o realizar acciones posteriores.
    * **`BEFORE UPDATE`:** Se ejecuta antes de actualizar una fila. Útil para validar los nuevos valores o modificar valores.
    * **`AFTER UPDATE`:** Se ejecuta después de actualizar una fila. Útil para auditar cambios o actualizar datos relacionados.
    * **`BEFORE DELETE`:** Se ejecuta antes de eliminar una fila. Útil para impedir la eliminación bajo ciertas condiciones o para registrar la eliminación.
    * **`AFTER DELETE`:** Se ejecuta después de eliminar una fila. Útil para limpiar datos relacionados en otras tablas o auditar la eliminación.

2.  **Triggers DDL (Data Definition Language):** Se activan por eventos relacionados con la definición de la estructura de la base de datos, como `CREATE`, `ALTER` o `DROP` de objetos (tablas, índices, vistas, etc.).
    * **Caso de uso:** Auditar quién y cuándo se creó o modificó una tabla.

3.  **Triggers de eventos de base de datos/sistema:** Se activan por eventos a nivel de la base de datos o de la sesión, como el inicio o cierre de la base de datos, inicio o cierre de sesión, errores, etc.
    * **Caso de uso:** Restringir el número de sesiones para un usuario específico al inicio de sesión.

4.  **`INSTEAD OF` Triggers (específico de algunos SGBD como SQL Server):** Permiten definir una acción alternativa que se ejecuta *en lugar* de la operación DML estándar en una vista. Esto es particularmente útil para permitir la modificación de datos a través de vistas que no son directamente actualizables.
    * **Caso de uso:** Permitir insertar datos en una vista que une varias tablas, distribuyendo la inserción en las tablas subyacentes.

## 6. ¿Cuándo ejecuta su acción un Trigger?

La ejecución de la acción de un trigger depende de su definición:

* **Triggers `BEFORE`:** El código del trigger se ejecuta **antes** de que la operación DML (INSERT, UPDATE, DELETE) real sobre la tabla se lleve a cabo. Esto significa que si el trigger modifica los valores `NEW`, esos serán los valores que finalmente se almacenen. Si el trigger lanza un error, la operación DML original se aborta y se revierte.
    * **Ejemplo:** Un trigger `BEFORE INSERT` que verifica la edad de una persona. Si la edad es menor de 18, lanza un error y la inserción no se realiza.

* **Triggers `AFTER`:** El código del trigger se ejecuta **después** de que la operación DML real sobre la tabla se ha completado exitosamente. En este punto, los cambios ya están reflejados en la tabla. Si el trigger `AFTER` lanza un error, la operación DML original también se revertirá (junto con cualquier cambio realizado por el trigger mismo, debido a la naturaleza transaccional).
    * **Ejemplo:** Un trigger `AFTER UPDATE` que registra el cambio de precio de un producto en una tabla de historial. El precio ya se ha actualizado en la tabla `Productos` antes de que se ejecute el código del trigger.

* **Triggers `INSTEAD OF` (para vistas):** El código del trigger se ejecuta **en lugar** de la operación DML sobre la vista. La operación DML original nunca se ejecuta. Es responsabilidad del código del trigger realizar las operaciones necesarias en las tablas base subyacentes para simular la operación deseada en la vista.
    * **Ejemplo:** Un trigger `INSTEAD OF INSERT` en una vista que combina `Clientes` y `Pedidos`. Cuando se intenta insertar una fila en la vista, el trigger inserta la información del cliente en la tabla `Clientes` y la información del pedido en la tabla `Pedidos`.

En resumen, la elección entre `BEFORE`, `AFTER` o `INSTEAD OF` es crucial y depende de la lógica de negocio y del momento en que se necesita intervenir en el flujo de una operación de base de datos.

# Ampliación de Triggers en Bases de Datos 📊

## 7. Ejemplos Detallados de Triggers 🛠️

### 7.1 Trigger de Auditoría en PostgreSQL
```sql
CREATE OR REPLACE FUNCTION log_salary_change()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.salary <> OLD.salary THEN
        INSERT INTO salary_audit (
            employee_id,
            old_salary,
            new_salary,
            changed_by,
            change_date
        ) VALUES (
            OLD.id,
            OLD.salary,
            NEW.salary,
            current_user,
            now()
        );
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER salary_audit_trigger
AFTER UPDATE ON employees
FOR EACH ROW EXECUTE FUNCTION log_salary_change();
```

**Caso de uso**: Cumplimiento normativo GDPR en RRHH

---

### 7.2 Trigger de Integridad Referencial
```sql
-- MySQL Example
DELIMITER //
CREATE TRIGGER prevent_orphan_records
BEFORE DELETE ON departments
FOR EACH ROW
BEGIN
    DECLARE employee_count INT;
    
    SELECT COUNT(*) INTO employee_count
    FROM employees
    WHERE department_id = OLD.id;
    
    IF employee_count > 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'No se puede eliminar: existen empleados en este departamento';
    END IF;
END//
DELIMITER ;
```

---

## 8. Trigger Marketing Avanzado 📧

### 8.1 Flujo Automatizado de Engagement

![Trigger Marketing](C:/Users/miaun/Downloads/TriggerMarketin8.png)

**Plataformas**:
- **Klaviyo**: Triggers basados en comportamiento
- **ActiveCampaign**: Automatizaciones multicanal
- **Braze**: Triggers en tiempo real

**Ejemplo real:**
```python
# Pseudocódigo para trigger de abandono de carrito
if user.cart_items and not user.purchase_within(24h):
    send_email(
        template="cart_reminder",
        items=user.cart_items,
        discount_code="RECUPERA10"
    )
    log_trigger_event("cart_abandoned", user.id)
```

---

## 9. Creación de Triggers Paso a Paso 📝

### 9.1 En SQL Server con Transacciones

```sql
CREATE TRIGGER trg_OrderAudit
ON Orders
AFTER INSERT, UPDATE, DELETE
AS
BEGIN
    SET NOCOUNT ON;
    
    BEGIN TRY
        BEGIN TRANSACTION;
        
        -- Lógica de auditoría
        INSERT INTO OrderAudit (...)
        SELECT ... FROM inserted;
        
        COMMIT TRANSACTION;
    END TRY
    BEGIN CATCH
        ROLLBACK TRANSACTION;
        -- Loggear error
        EXEC usp_LogError;
    END CATCH
END;
```

### 9.2 Buenas Prácticas

1. **Nomenclatura**: `trg_[Tabla]_[Acción]_[Propósito]`  
   Ej: `trg_Products_Insert_UpdateStock`

2. **Documentación**:
   ```sql
   /* 
   * Trigger: trg_Products_Update_PriceHistory
   * Propósito: Registrar cambios de precio para análisis
   * Creado: 2023-05-15
   * Dueño: DBA Team
   */
   ```

---

## 10. Características Clave de Triggers

| Característica | Implicación                         | Ejemplo                    |
|----------------|-------------------------------------|----------------------------|
| Transparente   | No requiere cambios en aplicación   | Auditoría automática       |
| Atómico        | Se ejecuta en la misma transacción  | Rollback si falla          |
| Contextual     | Accede al estado anterior/posterior | Comparar valores old/new   |
| Encadenable    | Puede activar otros triggers         | Actualización en cascada   |

![Diagrama Trigger](C:/Users/miaun/Downloads/diagramaPregunta10.png)

---

## 11. Riesgos y Alternativas a Triggers

### 11.1 Problemas Comunes

- **Recursión Indirecta**:
  ```sql
  -- Trigger A en Tabla1 → Actualiza Tabla2
  -- Trigger B en Tabla2 → Actualiza Tabla1
  ```

- **Problemas de Performance**:
  ```python
  # Antipatrón
  for _ in range(1_000_000):
      db.execute("INSERT INTO table VALUES(...)")
      # Se dispara trigger en cada inserción
  ```

### 11.2 Patrones de Mitigación

1. **Bulk Operations**:
   ```sql
   DISABLE TRIGGER trg_Audit ON Table;
   -- Operaciones masivas
   ENABLE TRIGGER trg_Audit ON Table;
   ```

2. **Event Sourcing**:
   ```javascript
   // En lugar de trigger
   eventStore.emit('userUpdated', {
       userId: 123,
       changes: {...}
   });
   ```

---

## 12. Alternativas Modernas por DB 🛡️

### 12.1 PostgreSQL

```sql
-- NOTIFY/LISTEN
CREATE TRIGGER notify_change
AFTER INSERT ON orders
FOR EACH ROW
EXECUTE FUNCTION pg_notify('order_created', NEW.id::text);
```

### 12.2 MongoDB Atlas Triggers

```json
{
  "name": "updateInventory",
  "type": "DATABASE",
  "config": {
    "operationTypes": ["insert"],
    "database": "ecommerce",
    "collection": "orders",
    "serviceName": "mongodb-atlas"
  },
  "function": "updateInventoryFunction"
}
```

---

### 12.3 Comparativa de Enfoques

| Enfoque     | Latencia | Complejidad | Escalabilidad |
|-------------|----------|-------------|----------------|
| Triggers    | Baja     | Media       | Limitada       |
| CDC         | Media    | Alta        | Alta           |
| Event Bus   | Alta     | Alta        | Muy Alta       |
| Polling     | Variable | Baja        | Media          |

> **Recomendación final**:  
> Usa triggers para lógica crítica sincrónica y eventos asincrónicos para operaciones complejas o distribuidas.
