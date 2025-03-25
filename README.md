-- Creación de la base de datos
CREATE DATABASE vtaszfs;
USE vtaszfs;

-- Tabla Clientes
CREATE TABLE Clientes (
id INT PRIMARY KEY AUTO_INCREMENT,
nombre VARCHAR(100),
email VARCHAR(100) UNIQUE
);

-- Tabla UbicacionCliente
CREATE TABLE UbicacionCliente (
id INT PRIMARY KEY AUTO_INCREMENT,
cliente_id INT,
direccion VARCHAR(255),
ciudad VARCHAR(100),
estado VARCHAR(50),
codigo_postal VARCHAR(10),
pais VARCHAR(50),
FOREIGN KEY (cliente_id) REFERENCES Clientes(id)
);

-- Tabla Empleados
CREATE TABLE Empleados (
id INT PRIMARY KEY AUTO_INCREMENT,
nombre VARCHAR(100),
puesto VARCHAR(50),
salario DECIMAL(10, 2),
fecha_contratacion DATE
);

-- Tabla Proveedores
CREATE TABLE Proveedores (
id INT PRIMARY KEY AUTO_INCREMENT,
nombre VARCHAR(100),
contacto VARCHAR(100),
telefono VARCHAR(20),
direccion VARCHAR(255)
);

-- Tabla TiposProductos
CREATE TABLE TiposProductos (
id INT PRIMARY KEY AUTO_INCREMENT,
tipo_nombre VARCHAR(100),
descripcion TEXT
);

-- Tabla Productos
CREATE TABLE Productos (
id INT PRIMARY KEY AUTO_INCREMENT,
nombre VARCHAR(100),
precio DECIMAL(10, 2),
proveedor_id INT,
tipo_id INT,
FOREIGN KEY (proveedor_id) REFERENCES Proveedores(id),
FOREIGN KEY (tipo_id) REFERENCES TiposProductos(id)
);

-- Tabla Pedidos
CREATE TABLE Pedidos (
id INT PRIMARY KEY AUTO_INCREMENT,
cliente_id INT,
fecha DATE,
total DECIMAL(10, 2),
FOREIGN KEY (cliente_id) REFERENCES Clientes(id)
);

-- Tabla DetallesPedido
CREATE TABLE DetallesPedido (
id INT PRIMARY KEY AUTO_INCREMENT,
pedido_id INT,
producto_id INT,
cantidad INT,
precio DECIMAL(10, 2),
FOREIGN KEY (pedido_id) REFERENCES Pedidos(id),
FOREIGN KEY (producto_id) REFERENCES Productos(id)
);

1. Normalización
     1. Crear una tabla HistorialPedidos que almacene cambios en los pedidos.
         CREATE TABLE HistorialPedidos (
            id INT PRIMARY KEY AUTO_INCREMENT,
            pedido_id INT,
            estado_anterior VARCHAR(50),
            estado_nuevo VARCHAR(50),
            fecha_cambio TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            FOREIGN KEY (pedido_id) REFERENCES Pedidos(id)
         );

     2. Evaluar la tabla Clientes para eliminar datos redundantes y normalizar hasta 3NF.
         CREATE TABLE ClientesDetalles (
            id INT PRIMARY KEY AUTO_INCREMENT,
            cliente_id INT,
            telefono VARCHAR(20),
            fecha_registro DATE,
            FOREIGN KEY (cliente_id) REFERENCES Clientes(id)
         );

     3. Separar la tabla Empleados en una tabla de DatosEmpleados y otra para Puestos.
         CREATE TABLE Puestos (
            id INT PRIMARY KEY AUTO_INCREMENT,
            nombre VARCHAR(50)
         );
         CREATE TABLE DatosEmpleados (
            id INT PRIMARY KEY AUTO_INCREMENT,
            nombre VARCHAR(100),
            puesto_id INT,
            salario DECIMAL(10, 2),
            fecha_contratacion DATE,
            FOREIGN KEY (puesto_id) REFERENCES Puestos(id)
         );
     4. Revisar la relación Clientes y UbicacionCliente para evitar duplicación de datos.
     5. Normalizar Proveedores para tener ContactoProveedores en otra tabla.
         CREATE TABLE ContactoProveedores (
            id INT PRIMARY KEY AUTO_INCREMENT,
            proveedor_id INT,
            contacto_nombre VARCHAR(100),
            telefono VARCHAR(20),
            email VARCHAR(100),
            FOREIGN KEY (proveedor_id) REFERENCES Proveedores(id)
         );
     6. Crear una tabla de Telefonos para almacenar múltiples números por cliente.
         CREATE TABLE TelefonosClientes (
            id INT PRIMARY KEY AUTO_INCREMENT,
            cliente_id INT,
            telefono VARCHAR(20),
            tipo_telefono VARCHAR(50),
            FOREIGN KEY (cliente_id) REFERENCES Clientes(id)
         );
     7. Transformar TiposProductos en una relación categórica jerárquica.
         ALTER TABLE TiposProductos
         ADD COLUMN parent_id INT;
     8. Normalizar Pedidos y DetallesPedido para evitar inconsistencias de precios.
         CREATE TABLE HistorialPreciosProductos (
            id INT PRIMARY KEY AUTO_INCREMENT,
            producto_id INT,
            precio DECIMAL(10, 2),
            fecha_inicio DATE,
            fecha_fin DATE,
            FOREIGN KEY (producto_id) REFERENCES Productos(id)
         );
     9. Usar una relación de muchos a muchos para Empleados y Proveedores.
         CREATE TABLE EmpleadoProveedor (
            empleado_id INT,
            proveedor_id INT,
            PRIMARY KEY (empleado_id, proveedor_id),
            FOREIGN KEY (empleado_id) REFERENCES Empleados(id),
            FOREIGN KEY (proveedor_id) REFERENCES Proveedores(id)
         );
     10. Convertir la tabla UbicacionCliente en una relación genérica de Ubicaciones .
         CREATE TABLE Ubicacion (
            id INT PRIMARY KEY AUTO_INCREMENT,
            entidad_id INT,
            entidad_tipo VARCHAR(50),
            direccion VARCHAR(100),
            ciudad VARCHAR(100),
            estado VARCHAR(50),
            codigo_postal VARCHAR(10),
            pais VARCHAR(50)
         );

         Estas columnas permitirán asociar cualquier entidad
             ALTER TABLE UbicacionCliente
             ADD COLUMN entidad_id INT,
             ADD COLUMN entidad_tipo VARCHAR(50);
        
        Agregaremos una relacion con la tabla Ubicacion
             ALTER TABLE UbicacionCliente
             ADD COLUMN ubicacion_id INT;
             
             ALTER TABLE UbicacionCliente
             ADD CONSTRAINT fk_ubicacion_id
             FOREIGN KEY (ubicacion_id) REFERENCES Ubicaciones(id);

2. Joins
     1. Obtener la lista de todos los pedidos con los nombres de clientes usando INNER JOIN .
         SELECT Pedidos.id AS pedido_id, Clientes.nombre AS cliente_nombre
         FROM Pedidos
         INNER JOIN Clientes ON Pedidos.cliente_id = Clientes.id;

     2. Listar los productos y proveedores que los suministran con INNER JOIN .
         SELECT Productos.nombre AS producto_nombre, Proveedores.nombre AS proveedor_nombre
         FROM Productos
         INNER JOIN Proveedores ON Productos.proveedor_id = Proveedores.id;

     3. Mostrar los pedidos y las ubicaciones de los clientes con LEFT JOIN .
         SELECT Pedidos.id AS pedido_id, Clientes.nombre AS cliente_nombre, Ubicacion.direccion AS ubicacion
         FROM Pedidos
         LEFT JOIN Clientes ON Pedidos.cliente_id = Clientes.id
         LEFT JOIN Ubicacion ON Clientes.id = Ubicacion.entidad_id AND Ubicacion.entidad_tipo = 'Cliente';

     4. Consultar los empleados que han registrado pedidos, incluyendo empleados sin pedidos ( LEFT JOIN ).
         SELECT Empleados.nombre AS empleado_nombre, Pedidos.id AS pedido_id
         FROM Empleados
         LEFT JOIN Pedidos ON Empleados.id = Pedidos.empleado_id;

     5. Obtener el tipo de producto y los productos asociados con INNER JOIN .
         SELECT TiposProductos.tipo_nombre, Productos.nombre AS producto_nombre
         FROM Productos
         INNER JOIN TiposProductos ON Productos.tipo_producto_id = TiposProductos.id;

     6. Listar todos los clientes y el número de pedidos realizados con COUNT y GROUP BY .
         SELECT Clientes.nombre AS cliente_nombre, COUNT(Pedidos.id) AS num_pedidos
         FROM Clientes
         LEFT JOIN Pedidos ON Clientes.id = Pedidos.cliente_id
         GROUP BY Clientes.id;

     7. Combinar Pedidos y Empleados para mostrar qué empleados gestionaron pedidos específicos.
         SELECT Pedidos.id AS pedido_id, Empleados.nombre AS empleado_nombre
         FROM Pedidos
         INNER JOIN Empleados ON Pedidos.empleado_id = Empleados.id;

     8. Mostrar productos que no han sido pedidos ( RIGHT JOIN ).
         SELECT Productos.nombre AS producto_nombre
         FROM Productos
         RIGHT JOIN DetallesPedido ON Productos.id = DetallesPedido.producto_id
         WHERE DetallesPedido.producto_id IS NULL;
         
     9. Mostrar el total de pedidos y ubicación de clientes usando múltiples JOIN .
         SELECT Clientes.nombre AS cliente_nombre, COUNT(Pedidos.id) AS total_pedidos, Ubicacion.direccion AS Ubicacion
         FROM Clientes
         LEFT JOIN Pedidos ON Clientes.id = Pedidos.cliente_id
         LEFT JOIN Ubicacion ON Clientes.id = Ubicacion.entidad_id AND Ubicaciones.entidad_tipo = 'Cliente'
         GROUP BY Clientes.id;

     10. Unir Proveedores , Productos , y TiposProductos para un listado completo de inventario.
         SELECT Proveedores.nombre AS proveedor_nombre, Productos.nombre AS producto_nombre, TiposProductos.tipo_nombre
         FROM Productos
         INNER JOIN Proveedores ON Productos.proveedor_id = Proveedores.id
         INNER JOIN TiposProductos ON Productos.tipo_producto_id = TiposProductos.id;

3. Consultas Simples
     1. Seleccionar todos los productos con precio mayor a $50.
         SELECT * 
         FROM Productos 
         WHERE precio > 50;
     2. Consultar clientes registrados en una ciudad específica.
         SELECT * 
         FROM Clientes 
         WHERE ciudad = 'NombreDeLaCiudad';
     3. Mostrar empleados contratados en los últimos 2 años.
         SELECT * 
         FROM Empleados 
         WHERE fecha_contrato >= CURDATE() - INTERVAL 2 YEAR;

     4. Seleccionar proveedores que suministran más de 5 productos.
         SELECT Proveedores.nombre
         FROM Proveedores
         JOIN Productos ON Proveedores.id = Productos.proveedor_id
         GROUP BY Proveedores.id
         HAVING COUNT(Productos.id) > 5;

     5. Listar clientes que no tienen dirección registrada en UbicacionCliente .
         SELECT Clientes.nombre 
         FROM Clientes
         LEFT JOIN UbicacionCliente ON Clientes.id = UbicacionCliente.cliente_id
         WHERE UbicacionCliente.direccion IS NULL;

     6. Calcular el total de ventas por cada cliente.
         SELECT Clientes.nombre, SUM(Ventas.monto) AS total_ventas
         FROM Ventas
         JOIN Clientes ON Ventas.cliente_id = Clientes.id
         GROUP BY Clientes.id;

     7. Mostrar el salario promedio de los empleados.
         SELECT AVG(salario) AS salario_promedio
         FROM Empleados;

     8. Consultar el tipo de productos disponibles en TiposProductos .
         SELECT tipo_nombre
         FROM TiposProductos;

     9. Seleccionar los 3 productos más caros.
         SELECT * 
         FROM Productos
         ORDER BY precio DESC
         LIMIT 3;

     10. Consultar el cliente con el mayor número de pedidos.
         SELECT Clientes.nombre, COUNT(Pedidos.id) AS num_pedidos
         FROM Clientes
         JOIN Pedidos ON Clientes.id = Pedidos.cliente_id
         GROUP BY Clientes.id
         ORDER BY num_pedidos DESC
         LIMIT 1;
4. Consultas Multitabla
     1. Listar todos los pedidos y el cliente asociado.
         SELECT p.id AS pedido_id, p.fecha, c.nombre AS cliente_nombre
         FROM Pedidos p
         JOIN Clientes c ON p.cliente_id = c.id;
         
     2. Mostrar la ubicación de cada cliente en sus pedidos.
         SELECT p.id AS pedido_id, c.nombre AS cliente_nombre, u.direccion AS ubicacion_cliente
         FROM Pedidos p
         JOIN Clientes c ON p.cliente_id = c.id
         JOIN UbicacionCliente u ON c.id = u.cliente_id;
         
     3. Listar productos junto con el proveedor y tipo de producto.
         SELECT pr.nombre AS producto_nombre, pr.precio, p.nombre AS proveedor_nombre, tp.tipo_nombre AS tipo_producto
         FROM Productos pr
         JOIN Proveedores p ON pr.proveedor_id = p.id
         JOIN TiposProductos tp ON pr.tipo_producto_id = tp.id;
         
     4. Consultar todos los empleados que gestionan pedidos de clientes en una ciudad específica.
         SELECT e.nombre AS empleado_nombre, c.nombre AS cliente_nombre, p.fecha AS fecha_pedido
         FROM Empleados e
         JOIN Pedidos p ON e.id = p.empleado_id
         JOIN Clientes c ON p.cliente_id = c.id
         WHERE c.ciudad = 'NombreDeLaCiudad';
         
     5. Consultar los 5 productos más vendidos.
         SELECT pr.nombre AS producto_nombre, SUM(dp.cantidad) AS total_vendido
         FROM DetallePedidos dp
         JOIN Productos pr ON dp.producto_id = pr.id
         GROUP BY pr.id
         ORDER BY total_vendido DESC
         LIMIT 5;
         
     6. Obtener la cantidad total de pedidos por cliente y ciudad.
         SELECT c.nombre AS cliente_nombre, c.ciudad, COUNT(p.id) AS total_pedidos
         FROM Clientes c
         JOIN Pedidos p ON c.id = p.cliente_id
         GROUP BY c.id, c.ciudad;
         
     7. Listar clientes y proveedores en la misma ciudad.
         SELECT c.nombre AS cliente_nombre, p.nombre AS proveedor_nombre, c.ciudad
         FROM Clientes c
         JOIN Pedidos pd ON c.id = pd.cliente_id
         JOIN Productos pr ON pd.id = pr.pedido_id
         JOIN Proveedores p ON pr.proveedor_id = p.id
         WHERE c.ciudad = p.ciudad;
         
     8. Mostrar el total de ventas agrupado por tipo de producto.
         SELECT tp.tipo_nombre, SUM(dp.cantidad * pr.precio) AS total_ventas
         FROM DetallePedidos dp
         JOIN Productos pr ON dp.producto_id = pr.id
         JOIN TiposProductos tp ON pr.tipo_producto_id = tp.id
         GROUP BY tp.tipo_nombre;
         
     9. Listar empleados que gestionan pedidos de productos de un proveedor específico.
         SELECT e.nombre AS empleado_nombre, p.nombre AS proveedor_nombre, COUNT(pd.id) AS total_pedidos
         FROM Empleados e
         JOIN Pedidos pd ON e.id = pd.empleado_id
         JOIN DetallePedidos dp ON pd.id = dp.pedido_id
         JOIN Productos pr ON dp.producto_id = pr.id
         JOIN Proveedores p ON pr.proveedor_id = p.id
         WHERE p.id = 'ProveedorID'
         GROUP BY e.id, p.id;

     10. Obtener el ingreso total de cada proveedor a partir de los productos vendidos.
         SELECT p.nombre AS proveedor_nombre, SUM(dp.cantidad * pr.precio) AS ingreso_total
         FROM DetallePedidos dp
         JOIN Productos pr ON dp.producto_id = pr.id
         JOIN Proveedores p ON pr.proveedor_id = p.id
         GROUP BY p.id;
         

5. Subconsultas
     1. Consultar el producto más caro en cada categoría.
         SELECT p.nombre, p.precio, p.tipo_producto_id
         FROM Productos p
         WHERE p.precio = (
             SELECT MAX(precio)
             FROM Productos
             WHERE tipo_producto_id = p.tipo_producto_id
         );

     2. Encontrar el cliente con mayor total en pedidos.
         SELECT c.nombre, SUM(p.monto) AS total_pedidos
         FROM Clientes c
         JOIN Pedidos p ON c.id = p.cliente_id
         GROUP BY c.id
         HAVING SUM(p.monto) = (
         SELECT MAX(total)
         FROM (
            SELECT cliente_id, SUM(monto) AS total
            FROM Pedidos
            GROUP BY cliente_id
            ) AS subquery
         );

     3. Listar empleados que ganan más que el salario promedio.
         SELECT * 
         FROM Empleados e
         WHERE e.salario > (
             SELECT AVG(salario) 
             FROM Empleados
         );

     4. Consultar productos que han sido pedidos más de 5 veces.
         SELECT p.nombre
         FROM Productos p
         JOIN DetallePedidos dp ON p.id = dp.producto_id
         GROUP BY p.id
         HAVING COUNT(dp.id) > 5;

     5. Listar pedidos cuyo total es mayor al promedio de todos los pedidos.
         SELECT * 
         FROM Pedidos p
         WHERE p.total > (
             SELECT AVG(total) 
             FROM Pedidos
         );
         
     6. Seleccionar los 3 proveedores con más productos.
         SELECT p.nombre, COUNT(pr.id) AS cantidad_productos
         FROM Proveedores p
         JOIN Productos pr ON p.id = pr.proveedor_id
         GROUP BY p.id
         ORDER BY cantidad_productos DESC
         LIMIT 3;
         
     7. Consultar productos con precio superior al promedio en su tipo.
         SELECT p.nombre, p.precio
         FROM Productos p
         WHERE p.precio > (
             SELECT AVG(precio)
             FROM Productos
             WHERE tipo_producto_id = p.tipo_producto_id
         );
         
     8. Mostrar clientes que han realizado más pedidos que la media.
         SELECT c.nombre, COUNT(p.id) AS cantidad_pedidos
         FROM Clientes c
         JOIN Pedidos p ON c.id = p.cliente_id
         GROUP BY c.id
         HAVING COUNT(p.id) > (
             SELECT AVG(cantidad)
             FROM (
                 SELECT COUNT(id) AS cantidad
                 FROM Pedidos
                 GROUP BY cliente_id
             ) AS subquery
         );
         
     9. Encontrar productos cuyo precio es mayor que el promedio de todos los productos.
         SELECT nombre, precio
         FROM Productos
         WHERE precio > (SELECT AVG(precio) FROM Productos);
         
     10. Mostrar empleados cuyo salario es menor al promedio del departamento.
         SELECT e.nombre, e.salario
         FROM Empleados e
         WHERE e.salario < (
             SELECT AVG(salario)
             FROM Empleados
             WHERE departamento_id = e.departamento_id
         );
         
6. Procedimientos Almacenados
     1. Crear un procedimiento para actualizar el precio de todos los productos de un proveedor.
         DELIMITER $$
         CREATE PROCEDURE actualizar_precio_proveedor(IN proveedor_id INT, IN nuevo_precio DECIMAL(10, 2))
         BEGIN
             UPDATE Productos
             SET precio = nuevo_precio
             WHERE proveedor_id = proveedor_id;
         END$$
         DELIMITER ;
         
     2. Un procedimiento que devuelva la dirección de un cliente por ID.
         DELIMITER $$
         CREATE PROCEDURE obtener_direccion_cliente(IN cliente_id INT)
         BEGIN
             SELECT direccion
             FROM UbicacionCliente
             WHERE cliente_id = cliente_id;
         END$$
         DELIMITER ;
         
     3. Crear un procedimiento que registre un pedido nuevo y sus detalles.
         DELIMITER $$
         CREATE PROCEDURE registrar_pedido(IN cliente_id INT, IN monto_total DECIMAL(10, 2))
         BEGIN
             DECLARE nuevo_pedido_id INT;
             INSERT INTO Pedidos (cliente_id, total, fecha) 
             VALUES (cliente_id, monto_total, NOW());
             SET nuevo_pedido_id = LAST_INSERT_ID();
         END$$
         DELIMITER ;
         
     4. Un procedimiento para calcular el total de ventas de un cliente.
         DELIMITER $$
         CREATE PROCEDURE calcular_total_ventas_cliente(IN cliente_id INT)
         BEGIN
             SELECT SUM(monto) AS total_ventas
             FROM Ventas
             WHERE cliente_id = cliente_id;
         END$$
         DELIMITER ;
         
     5. Crear un procedimiento para obtener los empleados por puesto.
         DELIMITER $$
         CREATE PROCEDURE obtener_empleados_por_puesto(IN puesto VARCHAR(50))
         BEGIN
             SELECT * 
             FROM Empleados
             WHERE puesto = puesto;
         END$$
         DELIMITER ;
         
     6. Un procedimiento que actualice el salario de empleados por puesto.
         DELIMITER $$        
         CREATE PROCEDURE actualizar_salario_por_puesto(IN puesto VARCHAR(50), IN nuevo_salario DECIMAL(10, 2))
         BEGIN
             UPDATE Empleados
             SET salario = nuevo_salario
             WHERE puesto = puesto;
         END$$
         DELIMITER ;

     7. Crear un procedimiento que liste los pedidos entre dos fechas.
         DELIMITER $$
         CREATE PROCEDURE listar_pedidos_entre_fechas(IN fecha_inicio DATE, IN fecha_fin DATE)
         BEGIN
             SELECT * 
             FROM Pedidos
             WHERE fecha BETWEEN fecha_inicio AND fecha_fin;
         END$$
         DELIMITER ;
         
     8. Un procedimiento para aplicar un descuento a productos de una categoría.
         DELIMITER $$
         CREATE PROCEDURE aplicar_descuento_categoria(IN categoria_id INT, IN porcentaje_descuento DECIMAL(5, 2))
         BEGIN
             UPDATE Productos
             SET precio = precio - (precio * porcentaje_descuento / 100)
             WHERE tipo_producto_id = categoria_id;
         END$$
         DELIMITER ;
         
     9. Crear un procedimiento que liste todos los proveedores de un tipo de producto.
         DELIMITER $$
         CREATE PROCEDURE listar_proveedores_tipo_producto(IN tipo_producto_id INT)
         BEGIN
             SELECT DISTINCT p.nombre
             FROM Proveedores p
             JOIN Productos pr ON p.id = pr.proveedor_id
             WHERE pr.tipo_producto_id = tipo_producto_id;
         END$$
         DELIMITER ;
         
     10. Un procedimiento que devuelva el pedido de mayor valor.
         DELIMITER $$
         CREATE PROCEDURE obtener_pedido_mayor_valor()
         BEGIN
             SELECT * 
             FROM Pedidos
             ORDER BY total DESC
             LIMIT 1;
         END$$
         DELIMITER ;

7. Funciones Definidas por el Usuario
     1. Crear una función que reciba una fecha y devuelva los días transcurridos.
         DELIMITER $$
         CREATE FUNCTION dias_transcurridos(fecha DATE) RETURNS INT
         BEGIN
             RETURN DATEDIFF(CURDATE(), fecha);
         END$$
         DELIMITER ;
         
     2. Crear una función para calcular el total con impuesto de un monto.
         DELIMITER $$
         CREATE FUNCTION calcular_impuesto(monto DECIMAL(10, 2), tasa_impuesto DECIMAL(5, 2)) RETURNS DECIMAL(10, 2)
         BEGIN
             RETURN monto + (monto * tasa_impuesto / 100);
         END$$
         DELIMITER ;
         
     3. Una función que devuelva el total de pedidos de un cliente específico.
         DELIMITER $$
         CREATE FUNCTION total_pedidos_cliente(cliente_id INT) RETURNS DECIMAL(10, 2)
         BEGIN
             DECLARE total DECIMAL(10, 2);
             SELECT SUM(total) INTO total
             FROM Pedidos
             WHERE cliente_id = cliente_id;
             RETURN total;
         END$$
         DELIMITER ;
         
     4. Crear una función para aplicar un descuento a un producto.
         DELIMITER $$
         CREATE FUNCTION aplicar_descuento_producto(precio DECIMAL(10, 2), descuento DECIMAL(5, 2)) RETURNS DECIMAL(10, 2)
         BEGIN
             RETURN precio - (precio * descuento / 100);
         END$$
         DELIMITER ;
         
     5. Una función que indique si un cliente tiene dirección registrada.
         DELIMITER $$
         CREATE FUNCTION tiene_direccion(cliente_id INT) RETURNS BOOLEAN
         BEGIN
             DECLARE direccion_existente BOOLEAN;
             SELECT COUNT(*) > 0 INTO direccion_existente
             FROM UbicacionCliente
             WHERE cliente_id = cliente_id AND direccion IS NOT NULL;
             RETURN direccion_existente;
         END$$
         DELIMITER ;
         
     6. Crear una función que devuelva el salario anual de un empleado.
         DELIMITER $$
         CREATE FUNCTION salario_anual(empleado_id INT) RETURNS DECIMAL(10, 2)
         BEGIN
             DECLARE salario DECIMAL(10, 2);
             SELECT salario INTO salario
             FROM Empleados
             WHERE id = empleado_id;
             RETURN salario * 12;
         END$$
         DELIMITER ;
         
     7. Una función para calcular el total de ventas de un tipo de producto.
         DELIMITER $$
         CREATE FUNCTION total_ventas_tipo_producto(tipo_producto_id INT) RETURNS DECIMAL(10, 2)
         BEGIN
             DECLARE total DECIMAL(10, 2);
             SELECT SUM(monto) INTO total
             FROM Ventas v
             JOIN Productos p ON v.producto_id = p.id
             WHERE p.tipo_producto_id = tipo_producto_id;
             RETURN total;
         END$$
         DELIMITER ;
         
     8. Crear una función para devolver el nombre de un cliente por ID.
         DELIMITER $$
         CREATE FUNCTION obtener_nombre_cliente(cliente_id INT) RETURNS VARCHAR(255)
         BEGIN
             DECLARE nombre_cliente VARCHAR(255);
             SELECT nombre INTO nombre_cliente
             FROM Clientes
             WHERE id = cliente_id;
             RETURN nombre_cliente;
         END$$
         DELIMITER ;
         
     9. Una función que reciba el ID de un pedido y devuelva su total.
         DELIMITER $$
         CREATE FUNCTION obtener_total_pedido(pedido_id INT) RETURNS DECIMAL(10, 2)
         BEGIN
             DECLARE total DECIMAL(10, 2);
             SELECT total INTO total
             FROM Pedidos
             WHERE id = pedido_id;
             RETURN total;
         END$$
         DELIMITER ;
         
     10. Crear una función que indique si un producto está en inventario.
         DELIMITER $$
         CREATE FUNCTION producto_en_inventario(producto_id INT) RETURNS BOOLEAN
         BEGIN
             DECLARE en_inventario BOOLEAN;
             SELECT COUNT(*) > 0 INTO en_inventario
             FROM Inventario
             WHERE producto_id = producto_id AND cantidad > 0;
             RETURN en_inventario;
         END$$
         DELIMITER ;
         
8. Triggers
     1. Crear un trigger que registre en HistorialSalarios cada cambio de salario de empleados.
         DELIMITER $$
         CREATE TRIGGER after_salario_update
         AFTER UPDATE ON Empleados
         FOR EACH ROW
         BEGIN
             INSERT INTO HistorialSalarios (empleado_id, salario_anterior,        salario_nuevo, fecha)
             VALUES (OLD.id, OLD.salario, NEW.salario, NOW());
         END$$
         DELIMITER ;
         
     2. Crear un trigger que evite borrar productos con pedidos activos.
         DELIMITER $$
         CREATE TRIGGER before_producto_delete
         BEFORE DELETE ON Productos
         FOR EACH ROW
         BEGIN
             DECLARE cnt INT;
             SELECT COUNT(*) INTO cnt
             FROM DetallePedidos dp
             WHERE dp.producto_id = OLD.id;
             IF cnt > 0 THEN
                 SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'No se puede eliminar el producto porque tiene pedidos activos';
             END IF;
         END$$
         DELIMITER ;
         
     3. Un trigger que registre en HistorialPedidos cada actualización en Pedidos .
         DELIMITER $$
         CREATE TRIGGER after_pedido_update
         AFTER UPDATE ON Pedidos
         FOR EACH ROW
         BEGIN
             INSERT INTO HistorialPedidos (pedido_id, total_anterior, total_nuevo, fecha)
             VALUES (OLD.id, OLD.total, NEW.total, NOW());
         END$$
         DELIMITER ;
         
     4. Crear un trigger que actualice el inventario al registrar un pedido.
         DELIMITER $$
         CREATE TRIGGER after_pedido_insert
         AFTER INSERT ON Pedidos
         FOR EACH ROW
         BEGIN
             DECLARE producto_id INT;
             DECLARE cantidad INT;
             <!-- -- Aquí iría el código para ajustar el inventario basado en el detalle del pedido -->
         END$$
         DELIMITER ;
         
     5. Un trigger que evite actualizaciones de precio a menos de $1.
         DELIMITER $$
         CREATE TRIGGER before_precio_update
         BEFORE UPDATE ON Productos
         FOR EACH ROW
         BEGIN
             IF NEW.precio < 1 THEN
                 SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'El precio no puede ser menor a $1';
             END IF;
         END$$
         DELIMITER ;
         
     6. Crear un trigger que registre la fecha de creación de un pedido en HistorialPedidos .
         DELIMITER $$
         CREATE TRIGGER after_pedido_insert
         AFTER INSERT ON Pedidos
         FOR EACH ROW
         BEGIN
             INSERT INTO HistorialPedidos (pedido_id, fecha_creacion)
             VALUES (NEW.id, NOW());
         END$$
         DELIMITER ;
         
     7. Un trigger que mantenga el precio total de cada pedido en Pedidos .
         DELIMITER $$
         CREATE TRIGGER after_detalle_pedido_insert
         AFTER INSERT ON DetallePedidos
         FOR EACH ROW
         BEGIN
             DECLARE total DECIMAL(10, 2);
             SELECT SUM(dp.cantidad * p.precio)
             INTO total
             FROM DetallePedidos dp
             JOIN Productos p ON dp.producto_id = p.id
             WHERE dp.pedido_id = NEW.pedido_id;
             UPDATE Pedidos
             SET total = total
             WHERE id = NEW.pedido_id;
         END$$
         DELIMITER ;
         
     8. Crear un trigger para validar que UbicacionCliente no esté vacío al crear un cliente.
         DELIMITER $$
         CREATE TRIGGER before_cliente_insert
         BEFORE INSERT ON Clientes
         FOR EACH ROW
         BEGIN
             IF NEW.direccion IS NULL OR TRIM(NEW.direccion) = '' THEN
                 SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'La dirección es obligatoria';
             END IF;
         END$$
         DELIMITER ;
         
     9. Un trigger que registre en LogActividades cada modificación en Proveedores .
         DELIMITER $$
         CREATE TRIGGER after_proveedor_update
         AFTER UPDATE ON Proveedores
         FOR EACH ROW
         BEGIN
             INSERT INTO LogActividades (tabla, id_modificado, fecha, accion)
             VALUES ('Proveedores', OLD.id, NOW(), 'Actualización');
         END$$
         DELIMITER ;
         
     10. Crear un trigger que registre en HistorialContratos cada cambio en Empleados .   
         DELIMITER $$
         CREATE TRIGGER after_empleado_update
         AFTER UPDATE ON Empleados
         FOR EACH ROW
         BEGIN
             INSERT INTO HistorialContratos (empleado_id, contrato_anterior, contrato_nuevo, fecha)
             VALUES (OLD.id, OLD.fecha_contrato, NEW.fecha_contrato, NOW());
         END$$
         DELIMITER ;
         