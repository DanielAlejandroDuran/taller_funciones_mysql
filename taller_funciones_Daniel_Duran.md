# **Taller de Funciones Definidas por el Usuario (UDFs) en MySQL** 🚀 -- Daniel Alejandro Duran Franco

## **📌 Introducción**

En este taller, desarrollarás varias funciones definidas por el usuario (UDFs) en MySQL para resolver problemas comunes en bases de datos. Aprenderás a crear funciones, utilizarlas en consultas y comprender en qué casos son útiles.

Cada ejercicio presenta un caso de estudio con datos y requerimientos específicos. ¡Ponte a prueba y mejora tus habilidades en SQL! 💡

## **🔹 Caso 1: Cálculo de Bonificaciones de Empleados**

### **Escenario**:

La empresa **TechCorp** otorga bonificaciones a sus empleados según su salario base. La bonificación se calcula así:

- Si el salario es menor a 2,000 USD → Bonificación del **10%**.
- Si el salario está entre 2,000 y 5,000 USD → Bonificación del **7%**.
- Si el salario es mayor a 5,000 USD → Bonificación del **5%**.

**Tarea**:

1. Crea una función llamada `calcular_bonificacion` que reciba un `DECIMAL(10,2)` con el salario y devuelva la bonificación correspondiente.

   ```sql
   DELIMITER $$
   
   CREATE FUNCTION calcular_bonificacion(salario DECIMAL(10,2))
   RETURNS DECIMAL(10,2)
   DETERMINISTIC
   READS SQL DATA
   BEGIN
       DECLARE bonificacion DECIMAL(10,2);
       
       IF salario < 2000 THEN
           SET bonificacion = salario * 0.10;
       ELSEIF salario >= 2000 AND salario <= 5000 THEN
           SET bonificacion = salario * 0.07;
       ELSE
           SET bonificacion = salario * 0.05;
       END IF;
       
       RETURN bonificacion;
   END$$
   
   DELIMITER ;
   ```



2. Usa la función en un `SELECT` sobre la tabla `empleados`.

   ```sql
   SELECT 
       id,
       nombre,
       salario,
       calcular_bonificacion(salario) AS bonificacion,
       (salario + calcular_bonificacion(salario)) AS salario_total
   FROM empleados
   ORDER BY salario;
   ```

   ```sql
   +----+-------------+---------+--------------+---------------+
   | id | nombre      | salario | bonificacion | salario_total |
   +----+-------------+---------+--------------+---------------+
   |  1 | Juan Pérez  | 1500.00 |       150.00 |       1650.00 |
   |  2 | Ana Gómez   | 3000.00 |       210.00 |       3210.00 |
   |  3 | Carlos Ruiz | 6000.00 |       300.00 |       6300.00 |
   +----+-------------+---------+--------------+---------------+
   ```



## **🔹 Caso 2: Cálculo de Edad de Clientes**

### **Escenario**:

En la empresa **MarketShop**, se necesita calcular la edad de los clientes a partir de su fecha de nacimiento para determinar estrategias de marketing.

**Tarea**:

1. Crea una función llamada `calcular_edad` que reciba un `DATE` y devuelva la edad del cliente.

   ```sql
   DELIMITER $$
   
   CREATE FUNCTION calcular_edad(fecha_nacimiento DATE)
   RETURNS INT
   DETERMINISTIC
   READS SQL DATA
   BEGIN
       DECLARE edad INT;
       
       SET edad = TIMESTAMPDIFF(YEAR, fecha_nacimiento, CURDATE());
       
       RETURN edad;
   END$$
   
   DELIMITER ;
   ```

2. Usa la función en un `SELECT` sobre la tabla `clientes`.

   ```sql
   SELECT 
       id,
       nombre,
       fecha_nacimiento,
       calcular_edad(fecha_nacimiento) AS edad_actual,
       CONCAT(calcular_edad(fecha_nacimiento), ' años') AS edad_txt
   FROM clientes
   ORDER BY fecha_nacimiento;
   ```

   ```sql
   +----+----------------+------------------+-------------+----------+
   | id | nombre         | fecha_nacimiento | edad_actual | edad_txt |
   +----+----------------+------------------+-------------+----------+
   |  2 | María López    | 1985-09-20       |          39 | 39 años  |
   |  1 | Luis Martínez  | 1990-06-15       |          35 | 35 años  |
   |  3 | Pedro Gómez    | 2000-03-10       |          25 | 25 años  |
   +----+----------------+------------------+-------------+----------+
   ```



## **🔹 Caso 3: Formatear Números de Teléfono**

### **Escenario**:

En **CallCenter Solutions**, los números de teléfono están almacenados sin formato y se necesita presentarlos en el formato `(XXX) XXX-XXXX`.

**Tarea**:

1. Crea una función llamada `formatear_telefono` que reciba un número de teléfono en formato `XXXXXXXXXX` y lo devuelva en formato `(XXX) XXX-XXXX`.

   **Tabla**

   ```sql
   CREATE TABLE contactos (
       id INT PRIMARY KEY AUTO_INCREMENT,
       nombre VARCHAR(50),
       telefono VARCHAR(20)
   );
   ```

   **Datos**

   ```sql
   INSERT INTO contactos (nombre, telefono) VALUES
   ('Ana García', '3001234567'),
   ('Luis Rodríguez', '3109876543'),
   ('María Fernández', '3205551234'),
   ('Carlos Martín', '3157778899'),
   ('Sofia Jiménez', '3123456789'),
   ('Pedro Sánchez', '3184567890'),
   ('Laura Torres', '3165432109'),
   ('Diego Morales', '3198765432');
   ```

   **Funcion**

   ```sql
   DELIMITER $
   
   CREATE FUNCTION formatear_telefono(telefono_raw VARCHAR(20))
   RETURNS VARCHAR(20)
   DETERMINISTIC
   BEGIN
       DECLARE telefono_limpio VARCHAR(20);
       
       SET telefono_limpio = REPLACE(REPLACE(REPLACE(telefono_raw, ' ', ''), '-', ''), '(', '');
       SET telefono_limpio = REPLACE(REPLACE(telefono_limpio, ')', ''), '.', '');
       
       IF LENGTH(telefono_limpio) = 10 THEN
           RETURN CONCAT('(', 
               SUBSTRING(telefono_limpio, 1, 3), ') ', 
               SUBSTRING(telefono_limpio, 4, 3), '-', 
               SUBSTRING(telefono_limpio, 7, 4));
       ELSE
           RETURN telefono_limpio;
       END IF;
   END$
   
   DELIMITER ;
   ```

2. Usa la función en un `SELECT` sobre la tabla `contactos`.

   ```sql
   SELECT 
       id,
       nombre,
       telefono as telefono_original,
       formatear_telefono(telefono) as telefono_formateado
   FROM contactos
   ORDER BY nombre;
   ```

   ```sql
   +----+-------------------+-------------------+---------------------+
   | id | nombre            | telefono_original | telefono_formateado |
   +----+-------------------+-------------------+---------------------+
   |  1 | Ana García        | 3001234567        | (300) 123-4567      |
   |  4 | Carlos Martín     | 3157778899        | (315) 777-8899      |
   |  8 | Diego Morales     | 3198765432        | (319) 876-5432      |
   |  7 | Laura Torres      | 3165432109        | (316) 543-2109      |
   |  2 | Luis Rodríguez    | 3109876543        | (310) 987-6543      |
   |  3 | María Fernández   | 3205551234        | (320) 555-1234      |
   |  6 | Pedro Sánchez     | 3184567890        | (318) 456-7890      |
   |  5 | Sofia Jiménez     | 3123456789        | (312) 345-6789      |
   +----+-------------------+-------------------+---------------------+
   ```



## **🔹 Caso 4: Clasificación de Productos por Precio**

### **Escenario**:

En la tienda **E-Shop**, los productos se categorizan en tres niveles según su precio:

- **Bajo**: Menos de 50 USD.
- **Medio**: Entre 50 y 200 USD.
- **Alto**: Más de 200 USD.

**Tarea**:

1. Crea una función llamada `clasificar_precio` que reciba un `DECIMAL(10,2)` y devuelva un `VARCHAR(10)` con la clasificación del producto (`Bajo`, `Medio`, `Alto`).

   **Tabla**

   ```sql
   CREATE TABLE productos (
       id INT PRIMARY KEY AUTO_INCREMENT,
       nombre VARCHAR(100),
       precio DECIMAL(10,2),
       categoria VARCHAR(50)
   );
   ```

   **Datos**

   ```sql
   INSERT INTO productos (nombre, precio, categoria) VALUES
   ('Auriculares Bluetooth', 25.99, 'Electrónicos'),
   ('Camiseta Básica', 15.50, 'Ropa'),
   ('Smartphone Pro', 899.99, 'Electrónicos'),
   ('Zapatos Deportivos', 75.00, 'Calzado'),
   ('Laptop Gaming', 1299.99, 'Electrónicos'),
   ('Libro de Cocina', 12.99, 'Libros'),
   ('Reloj Inteligente', 199.99, 'Electrónicos'),
   ('Pantalón Jeans', 45.00, 'Ropa'),
   ('Cafetera Espresso', 150.00, 'Hogar'),
   ('Mochila Viaje', 89.99, 'Accesorios');
   ```

   **Funcion**

   ```sql
   DELIMITER $$
   
   CREATE FUNCTION clasificar_precio(precio DECIMAL(10,2))
   RETURNS VARCHAR(10)
   DETERMINISTIC
   BEGIN
       DECLARE clasificacion VARCHAR(10);
       
       IF precio < 50 THEN
           SET clasificacion = 'Bajo';
       ELSEIF precio >= 50 AND precio <= 200 THEN
           SET clasificacion = 'Medio';
       ELSE
           SET clasificacion = 'Alto';
       END IF;
       
       RETURN clasificacion;
   END$$
   
   DELIMITER ;
   ```

2. Usa la función en un `SELECT` sobre la tabla `productos`.

   ```sql
   SELECT 
       id,
       nombre,
       precio,
       categoria,
       clasificar_precio(precio) as clasificacion_precio
   FROM productos
   ORDER BY precio;
   ```

   ```sql
   +----+-----------------------+---------+---------------+----------------------+
   | id | nombre                | precio  | categoria     | clasificacion_precio |
   +----+-----------------------+---------+---------------+----------------------+
   |  6 | Libro de Cocina       |   12.99 | Libros        | Bajo                 |
   |  2 | Camiseta Básica       |   15.50 | Ropa          | Bajo                 |
   |  1 | Auriculares Bluetooth |   25.99 | Electrónicos  | Bajo                 |
   |  8 | Pantalón Jeans        |   45.00 | Ropa          | Bajo                 |
   |  4 | Zapatos Deportivos    |   75.00 | Calzado       | Medio                |
   | 10 | Mochila Viaje         |   89.99 | Accesorios    | Medio                |
   |  9 | Cafetera Espresso     |  150.00 | Hogar         | Medio                |
   |  7 | Reloj Inteligente     |  199.99 | Electrónicos  | Medio                |
   |  3 | Smartphone Pro        |  899.99 | Electrónicos  | Alto                 |
   |  5 | Laptop Gaming         | 1299.99 | Electrónicos  | Alto                 |
   +----+-----------------------+---------+---------------+----------------------+
   ```

   



