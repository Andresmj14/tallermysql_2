# tallermysql_2


# 📄 Consultas y Procedimientos SQL

## 📌 Consultas

### 1️⃣ Producto más caro (top 3)

```sql
SELECT descripcion_p, precio AS productos_caros 
FROM productos 
ORDER BY precio DESC 
LIMIT 3;
```

---

### 2️⃣ Cliente con mayor cantidad de pedidos

```sql
SELECT c.nombre AS cliente, COUNT(p.id) AS mayoria_pedidos 
FROM clientes c 
JOIN pedidos p ON c.id = p.id_cliente 
GROUP BY c.id, c.nombre 
LIMIT 1;
```

---

### 3️⃣ Empleados con salario mayor al promedio

```sql
SELECT nombre, salario 
FROM empleados 
WHERE salario > (SELECT AVG(salario) FROM empleados);
```

---

### 4️⃣ Productos pedidos más de 5 veces

```sql
SELECT 
    p.producto_id, 
    p.nombre_producto,
    COUNT(*) AS veces_pedido
FROM 
    detalles_pedido dp
JOIN 
    productos p 
ON 
    dp.producto_id = p.producto_id
GROUP BY 
    p.producto_id, p.nombre_producto
HAVING 
    COUNT(*) > 5;
```

---

### 5️⃣ Pedidos con total superior al promedio

```sql
SELECT id, total 
FROM pedidos
WHERE total > (SELECT AVG(total) FROM pedidos);
```

---

### 6️⃣ Proveedores con mayor cantidad de productos

```sql
SELECT pr.id AS proveedor_id, pr.dato_proveedores AS nombre_proveedor, COUNT(p.id_producto) AS total_productos 
FROM proveedores pr 
JOIN productos p ON pr.id = p.proveedor_id 
GROUP BY pr.id, pr.dato_proveedores 
ORDER BY total_productos DESC 
LIMIT 3;
```

---

### 7️⃣ Productos con precio superior al promedio de su categoría

```sql
SELECT p.id_producto, p.descripcion_p, p.precio, p.id_tp 
FROM productos p
WHERE p.precio > (
    SELECT AVG(p2.precio) 
    FROM productos p2 
    WHERE p2.id_tp = p.id_tp
);
```

---

### 8️⃣ Clientes con total de pedidos superior al promedio

```sql
SELECT 
    c.id AS id_cliente,
    CONCAT(c.nombre, ' ', c.apellido) AS nombre_completo,
    COUNT(p.id_pedido) AS total_pedidos
FROM 
    clientes c
JOIN 
    pedidos p ON c.id = p.id_cliente
GROUP BY 
    c.id, c.nombre, c.apellido
HAVING 
    COUNT(p.id_pedido) > (
        SELECT AVG(total_pedidos)
        FROM (
            SELECT 
                COUNT(*) AS total_pedidos
            FROM 
                pedidos
            GROUP BY 
                id_cliente
        ) AS subquery
    );
```

---

### 9️⃣ Productos con precio superior al promedio global

```sql
SELECT 
    id_producto,
    descripcion_p,
    precio
FROM 
    productos
WHERE 
    precio > (
        SELECT AVG(precio) 
        FROM productos
    );
```

---

### 🔟 Empleados con salario inferior al promedio

```sql
SELECT id_empleado, nombre_empleado, salario 
FROM datos_empleados 
WHERE salario < (SELECT AVG(salario) FROM empleados);
```

---

## 🛠️ Procedimientos almacenados

### ✅ Actualizar precios

```sql
SELECT descripcion_p, precio, proveedor_id 
FROM productos;

CALL actualizar_precios(1, 5);
```

---

### ✅ Obtener dirección de cliente

```sql
DELIMITER //

CREATE PROCEDURE obtener_direccion_cliente (
    IN p_id_cliente INT,
    OUT p_direccion VARCHAR(200)
)
BEGIN
    SELECT direccion
    INTO p_direccion
    FROM clientes
    WHERE id_cliente = p_id_cliente;
END //

DELIMITER ;

SET @direccion_cliente = '';

CALL obtener_direccion_cliente(5, @direccion_cliente);

SELECT @direccion_cliente;
```

---

### ✅ Registrar pedido con dos productos

```sql
DELIMITER //

CREATE PROCEDURE registrar_pedido (
    IN p_id_cliente INT,
    IN p_fecha DATE,
    IN p_id_producto1 INT,
    IN p_cantidad1 INT,
    IN p_precio_unitario1 DECIMAL(10,2),
    IN p_id_producto2 INT,
    IN p_cantidad2 INT,
    IN p_precio_unitario2 DECIMAL(10,2)
)
BEGIN
    DECLARE v_id_pedido INT;

    INSERT INTO pedidos (id_cliente, fecha_pedido)
    VALUES (p_id_cliente, p_fecha);
    
    SET v_id_pedido = LAST_INSERT_ID();
    
    INSERT INTO detalles_pedido (id_pedido, id_producto, cantidad, precio_unitario)
    VALUES (v_id_pedido, p_id_producto1, p_cantidad1, p_precio_unitario1);

    INSERT INTO detalles_pedido (id_pedido, id_producto, cantidad, precio_unitario)
    VALUES (v_id_pedido, p_id_producto2, p_cantidad2, p_precio_unitario2);

END //

DELIMITER ;

CALL registrar_pedido(
    1,                 
    CURDATE(),          
    10,                  
    12.50,               
    15,                  
    2,                   
    25.00                
);
```

---

### ✅ Calcular total de ventas de un cliente

```sql
DELIMITER //

CREATE PROCEDURE calcular_total_ventas_cliente (
    IN p_id_cliente INT,
    OUT p_total DECIMAL(10,2)
)
BEGIN
    SELECT SUM(dp.cantidad * dp.precio_unitario)
    INTO p_total
    FROM pedidos p
    INNER JOIN detalles_pedido dp ON p.id_pedido = dp.id_pedido
    WHERE p.id_cliente = p_id_cliente;

    IF p_total IS NULL THEN
        SET p_total = 0;
    END IF;
END //

DELIMITER ;

SET @total = 0;

CALL calcular_total_ventas_cliente(3, @total);

SELECT @total AS total_ventas;
```

---

### ✅ Obtener empleados por puesto

```sql
DELIMITER //

CREATE PROCEDURE obtener_empleados_por_puesto (
    IN p_id_puesto INT
)
BEGIN
    SELECT e.id_empleado, e.nombre_empleado, p.nombre_puesto
    FROM empleados e
    INNER JOIN puestos p ON e.id_puesto = p.id_puesto
    WHERE e.id_puesto = p_id_puesto;
END //

DELIMITER ;

CALL obtener_empleados_por_puesto(2);
```

---

### ✅ Actualizar salario por puesto

```sql
DELIMITER //

CREATE PROCEDURE actualizar_salario_por_puesto (
    IN p_id_puesto INT,
    IN p_nuevo_salario DECIMAL(10,2)
)
BEGIN
    UPDATE empleados
    SET salario = p_nuevo_salario
    WHERE id_puesto = p_id_puesto;
END //

DELIMITER ;

CALL actualizar_salario_por_puesto(3, 2500.00);
```

---

### ✅ Listar pedidos entre fechas

```sql
DELIMITER //

CREATE PROCEDURE listar_pedidos_entre_fechas (
    IN p_fecha_inicio DATE,
    IN p_fecha_fin DATE
)
BEGIN
    SELECT id_pedido, id_cliente, fecha_pedido, total
    FROM pedidos
    WHERE fecha_pedido BETWEEN p_fecha_inicio AND p_fecha_fin
    ORDER BY fecha_pedido ASC;
END //

DELIMITER ;

CALL listar_pedidos_entre_fechas('2025-06-01', '2025-06-30');
```

---

### ✅ Aplicar descuento por categoría

```sql
DELIMITER //

CREATE PROCEDURE aplicar_descuento_categoria (
    IN p_id_categoria INT,
    IN p_descuento_porcentaje DECIMAL(5,2)
)
BEGIN
    UPDATE productos
    SET precio = precio * (1 - p_descuento_porcentaje / 100)
    WHERE id_categoria = p_id_categoria;
END //

DELIMITER ;

CALL aplicar_descuento_categoria(3, 15);
```

---
