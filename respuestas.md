# Respuestas — Práctica SQL

## Pregunta 1 — Catálogo comercial activo

```sql
SELECT product_name AS producto, ROUND(unit_price::numeric) AS precio FROM products
WHERE discontinued = 0 AND unit_price BETWEEN 10 AND 50
ORDER BY precio DESC;
```

![Resultado de la pregunta 1](1.PNG)

## Pregunta 2 — Concentración geográfica de la cartera

```sql
SELECT country AS pais, COUNT(*) AS num_clientes, COUNT(DISTINCT city) AS num_ciudades FROM customers
GROUP BY country HAVING COUNT(*) >= 5
ORDER BY num_clientes DESC;
```

![Resultado de la pregunta 2](2.PNG)

## Pregunta 3 — Alerta de reposición

```sql
SELECT product_name AS producto, units_in_stock AS stock, reorder_level AS nivel_reposicion, 
	units_on_order AS pedido_a_proveedor,
CASE WHEN units_in_stock = 0 THEN 'CRÍTICO' ELSE 'AVISO' END AS situacion FROM products
WHERE discontinued = 0 AND units_in_stock <= reorder_level;
```

![Resultado de la pregunta 3](3.PNG)


## Pregunta 4 — Ficha completa de producto

```sql
SELECT p.product_name AS producto, c.category_name AS categoria, s.company_name AS proveedor, s.country AS pais, s.city AS ciudad FROM products p
INNER JOIN categories c ON p.category_id = c.category_id
INNER JOIN suppliers s ON p.supplier_id = s.supplier_id
WHERE s.country IN ('Italy', 'France', 'Spain') ORDER BY pais, producto;
```

![Resultado de la pregunta 4](4.PNG)

## Pregunta 5 — Detalle valorizado de un pedido

```sql
SELECT c.company_name AS cliente, ord.order_date AS fecha_pedido, p.product_name AS producto,
	ROUND(od.unit_price::numeric, 2) AS precio_unitario, od.quantity AS cantidad, od.discount AS descuento,
	ROUND(od.unit_price::numeric * od.quantity * (1 - od.discount::numeric), 2) AS importe_linea FROM orders ord
INNER JOIN order_details od USING (order_id)
INNER JOIN products p USING (product_id)
INNER JOIN customers c USING (customer_id)
WHERE ord.order_id = 10248;
```

![Resultado de la pregunta 5](5.PNG)

## Pregunta 6 — Ranking de categorías por facturación

```sql
SELECT c.category_name AS categoria, COUNT(*) AS num_lineas, COUNT(DISTINCT p.product_id) AS num_productos,
    ROUND(SUM(od.unit_price::numeric * od.quantity * (1 - od.discount::numeric)), 2) AS facturacion FROM order_details od
INNER JOIN products p ON od.product_id = p.product_id
INNER JOIN categories c ON p.category_id = c.category_id
GROUP BY c.category_name
HAVING SUM(od.unit_price::numeric * od.quantity * (1 - od.discount::numeric)) > 100000
ORDER BY facturacion DESC;
```

![Resultado de la pregunta 6](6.PNG)

## Pregunta 7 — Clientes sin actividad comercial

```sql
SELECT c.company_name AS cliente, c.country AS pais, COUNT(o.order_id) AS num_pedidos,
    COALESCE(MAX(o.order_date)::text, 'SIN PEDIDOS') AS ultimo_pedido FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.company_name, c.country
ORDER BY COUNT(o.order_id) ASC, cliente ASC;
```

![Resultado de la pregunta 7](7.PNG)

## Pregunta 8 — Organigrama de la fuerza de ventas

```sql
SELECT emp.first_name || ' ' || emp.last_name AS empleado, emp.title AS cargo,
    COALESCE(jefe.first_name || ' ' || jefe.last_name, 'DIRECCIÓN GENERAL') AS responsable,
    jefe.title AS cargo_responsable FROM employees emp
LEFT JOIN employees jefe ON emp.reports_to = jefe.employee_id
ORDER BY empleado;
```

![Resultado de la pregunta 8](8.PNG)

## Pregunta 9 — Rejilla de cobertura categoría × año

```sql
SELECT c.category_name AS categoria, a.anio, ROUND(COALESCE(v.facturacion, 0), 2) AS facturacion FROM categories c
CROSS JOIN (VALUES (1996), (1997), (1998)) AS a(anio)
LEFT JOIN (
	SELECT p.category_id, EXTRACT(YEAR FROM o.order_date)::int AS anio,
        SUM(od.unit_price::numeric * od.quantity * (1 - od.discount::numeric)) AS facturacion
    FROM orders o
    INNER JOIN order_details od USING (order_id)
    INNER JOIN products p USING (product_id)
    GROUP BY p.category_id, EXTRACT(YEAR FROM o.order_date)) v ON v.category_id = c.category_id
   		AND v.anio = a.anio
ORDER BY categoria, anio;
```

![Resultado de la pregunta 9](9.PNG)

## Pregunta 10 — Mapa de países: clientes frente a proveedores

```sql
SELECT
    COALESCE(c.pais, p.pais) AS pais,
    COALESCE(c.num_clientes, 0) AS num_clientes,
    COALESCE(p.num_proveedores, 0) AS num_proveedores,
    CASE
        WHEN c.pais IS NOT NULL AND p.pais IS NOT NULL THEN 'AMBOS'
        WHEN c.pais IS NOT NULL THEN 'SOLO CLIENTES'
        ELSE 'SOLO PROVEEDORES'
    END AS tipo_presencia
FROM (SELECT country AS pais, COUNT(*) AS num_clientes FROM customers GROUP BY country) c
FULL JOIN (SELECT country AS pais, COUNT(*) AS num_proveedores FROM suppliers GROUP BY country) p ON c.pais = p.pais
ORDER BY pais;
```

![Resultado de la pregunta 10](10.PNG)

## Pregunta 11 — Directorio unificado de contactos

```sql
SELECT 'CLIENTE' AS origen, UPPER(contact_name) AS contacto, company_name AS organizacion, city AS ciudad, country AS pais FROM customers
UNION ALL
SELECT 'PROVEEDOR' AS origen, UPPER(contact_name) AS contacto, company_name AS organizacion, city AS ciudad, country AS pais FROM suppliers
UNION ALL
SELECT 'EMPLEADO' AS origen, UPPER(first_name || ' ' || last_name) AS contacto, 'NORTHWIND TRADERS' AS organizacion, city AS ciudad,
    country AS pais FROM employees
ORDER BY origen, pais;
```

![Resultado de la pregunta 11](11.PNG)

## Pregunta 12 — Mercados con desequilibrio

### a) Países con clientes pero sin proveedores

```sql
-- Países con clientes pero sin ningún proveedor
SELECT country AS pais FROM customers
EXCEPT
SELECT country AS pais FROM suppliers
ORDER BY pais;
```

![Resultado de la pregunta 12a](12a.PNG)

### b) Países con clientes y proveedores

```sql
-- Países donde existen tanto clientes como proveedores
SELECT country AS pais FROM customers
INTERSECT
SELECT country AS pais FROM suppliers
ORDER BY pais;
```

![Resultado de la pregunta 12b](12b.PNG)

## Pregunta 13 — Clientes que nunca han comprado pescado

```sql
SELECT c.company_name AS cliente, c.country AS pais, COUNT(o.order_id) AS pedidos_realizados FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE NOT EXISTS (
    SELECT 1 FROM orders o2
    INNER JOIN order_details od USING (order_id)
    INNER JOIN products p USING (product_id)
    INNER JOIN categories cat USING (category_id)
    WHERE o2.customer_id = c.customer_id AND cat.category_name = 'Seafood'
)
GROUP BY c.customer_id, c.company_name, c.country
ORDER BY pedidos_realizados DESC, cliente;
```

![Resultado de la pregunta 13](13.PNG)

## Pregunta 14 — Productos por encima de la media

```sql
SELECT product_name AS producto,
    ROUND(unit_price::numeric, 2) AS precio,
    ROUND((SELECT AVG(unit_price::numeric) FROM products), 2) AS precio_medio_catalogo,
    ROUND(unit_price::numeric - (SELECT AVG(unit_price::numeric) FROM products), 2) AS diferencia
FROM products
WHERE discontinued = 0 AND unit_price::numeric > (SELECT AVG(unit_price::numeric)FROM products)
ORDER BY diferencia DESC;
```

![Resultado de la pregunta 14](14.PNG)

## Pregunta 15 — Ticket medio por cliente

```sql
SELECT c.company_name AS cliente, c.country AS pais, COUNT(p.order_id) AS num_pedidos,
    ROUND(SUM(p.importe_pedido), 2) AS importe_total,
    ROUND(AVG(p.importe_pedido), 2) AS ticket_medio
FROM customers c
INNER JOIN (
    SELECT o.order_id, o.customer_id,
        SUM(ROUND(d.unit_price::numeric * d.quantity * (1 - d.discount::numeric), 2)) AS importe_pedido
    FROM orders o
    INNER JOIN order_details d USING (order_id)
    GROUP BY o.order_id, o.customer_id
) p ON c.customer_id = p.customer_id
GROUP BY c.customer_id, c.company_name, c.country
ORDER BY ticket_medio DESC LIMIT 15;
```

![Resultado de la pregunta 15](15.PNG)

## Pregunta 16 — El producto más caro de cada categoría

```sql
SELECT c.category_name AS categoria, p.product_name AS producto,
    ROUND(p.unit_price::numeric, 2) AS precio,
    ROUND((SELECT AVG(p2.unit_price::numeric) FROM products p2 WHERE p2.category_id = p.category_id), 2) AS precio_medio_categoria
FROM products p
INNER JOIN categories c ON p.category_id = c.category_id
WHERE p.unit_price = (SELECT MAX(p3.unit_price) FROM products p3 WHERE p3.category_id = p.category_id)
ORDER BY categoria;
```

![Resultado de la pregunta 16](16.PNG)

## Pregunta 17 — Segmentación ABC de la cartera de clientes

```sql
WITH facturacion_clientes AS (
    SELECT c.customer_id, c.company_name,
        SUM(ROUND(od.unit_price::numeric * od.quantity * (1 - od.discount::numeric), 2)) AS facturacion_cliente
    FROM customers c
    INNER JOIN orders o USING (customer_id)
    INNER JOIN order_details od USING (order_id)
    GROUP BY c.customer_id, c.company_name
),

cuartiles AS (
    SELECT customer_id, company_name, facturacion_cliente, 
		NTILE(4) OVER (ORDER BY facturacion_cliente DESC) AS cuartil
    FROM facturacion_clientes
),

segmentados AS (
    SELECT customer_id, company_name, facturacion_cliente, cuartil,
        CASE
            WHEN cuartil = 1 THEN 'A - Estratégico'
            WHEN cuartil = 2 THEN 'B - Consolidado'
            WHEN cuartil = 3 THEN 'C - Ocasional'
            ELSE 'D - Marginal'
        END AS segmento
    FROM cuartiles
),

total_compania AS (
	SELECT SUM(facturacion_cliente) AS facturacion_total FROM facturacion_clientes
)

SELECT s.segmento, COUNT(*) AS num_clientes,
    ROUND(SUM(s.facturacion_cliente), 2) AS facturacion_segmento,
    ROUND(SUM(s.facturacion_cliente)/t.facturacion_total * 100, 2) AS porcentaje_sobre_total
FROM segmentados s
CROSS JOIN total_compania t
GROUP BY s.segmento, t.facturacion_total
ORDER BY segmento;
```

![Resultado de la pregunta 17](17.PNG)

## Pregunta 18 — Los tres productos más vendidos de cada categoría

```sql
WITH ventas_producto AS (
    SELECT c.category_name AS categoria, p.product_id, p.product_name AS producto,
        SUM(od.quantity) AS unidades,
        SUM(ROUND(od.unit_price::numeric * od.quantity * (1 - od.discount::numeric), 2)) AS facturacion
    FROM order_details od
    INNER JOIN products p USING (product_id)
    INNER JOIN categories c USING (category_id)
    GROUP BY c.category_name, p.product_id, p.product_name
),

ranking AS (
    SELECT categoria, product_id, producto, unidades, facturacion,
        ROW_NUMBER() OVER (PARTITION BY categoria ORDER BY facturacion DESC, producto ASC) AS posicion_en_categoria,
        RANK() OVER (ORDER BY facturacion DESC) AS posicion_global
    FROM ventas_producto
)

SELECT categoria, posicion_en_categoria, producto, unidades,
    ROUND(facturacion, 2) AS facturacion,
    posicion_global
FROM ranking
WHERE posicion_en_categoria <= 3
ORDER BY categoria, posicion_en_categoria;
```

![Resultado de la pregunta 18](18.PNG)

## Pregunta 19 — Evolución mensual con acumulado y media móvil

```sql
WITH facturacion_mensual AS (
    SELECT
        DATE_TRUNC('month', o.order_date)::date AS mes,
        SUM(ROUND(od.unit_price::numeric * od.quantity * (1 - od.discount::numeric), 2)) AS facturacion
    FROM orders o
    INNER JOIN order_details od USING (order_id)
    WHERE o.order_date >= '1997-01-01' AND o.order_date < '1998-01-01'
    GROUP BY DATE_TRUNC('month', o.order_date)
),

evolucion AS (
    SELECT mes, facturacion,
		SUM(facturacion) OVER (ORDER BY mes) AS acumulado,
		AVG(facturacion) OVER (ORDER BY mes ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS media_movil_3m,
		LAG(facturacion) OVER (ORDER BY mes) AS mes_anterior
	FROM facturacion_mensual
)

SELECT mes,
    ROUND(facturacion, 2) AS facturacion,
    ROUND(acumulado, 2) AS acumulado,
    ROUND(media_movil_3m, 2) AS media_movil_3m,
    ROUND(mes_anterior, 2) AS mes_anterior,
    ROUND((facturacion - mes_anterior)/NULLIF(mes_anterior, 0) * 100, 2) AS variacion_pct
FROM evolucion
ORDER BY mes;
```

![Resultado de la pregunta 19](19.PNG)

## Pregunta 20 — Cuadro de mando anual por categoría

```sql
-- Cuadro anual por categoría.
-- IMPORTANTE: 1996 solo contiene datos desde julio y 1998 hasta mayo,
-- por lo que la comparación 1997-1998 no representa años completos
-- y la tendencia debe interpretarse con cautela.

WITH ventas AS (
    SELECT c.category_name AS categoria,
        EXTRACT(YEAR FROM o.order_date)::int AS anio,
        ROUND(od.unit_price::numeric * od.quantity * (1 - od.discount::numeric), 2) AS importe
    FROM orders o
    INNER JOIN order_details od USING (order_id)
    INNER JOIN products p USING (product_id)
    INNER JOIN categories c USING (category_id)
),

resumen AS (
    SELECT categoria,
        GROUPING(categoria) AS es_total,
		COALESCE(SUM(importe) FILTER (WHERE anio = 1996), 0) AS f_1996,
		COALESCE(SUM(importe) FILTER (WHERE anio = 1997), 0) AS f_1997,
		COALESCE(SUM(importe) FILTER (WHERE anio = 1998), 0) AS f_1998,
		SUM(importe) AS total
    FROM ventas
    GROUP BY ROLLUP(categoria)
),

con_total AS (
    SELECT *,
        MAX(total) FILTER (WHERE es_total = 1) OVER () AS total_compania
    FROM resumen
)

SELECT
    CASE
        WHEN es_total = 1 THEN 'TOTAL'
        ELSE categoria
    END AS categoria,

    ROUND(f_1996, 2) AS f_1996,
    ROUND(f_1997, 2) AS f_1997,
    ROUND(f_1998, 2) AS f_1998,
    ROUND(total, 2) AS total,
	ROUND(total / total_compania * 100, 2) AS peso_pct,

    CASE
        WHEN es_total = 1 THEN 'TOTAL GENERAL'
        WHEN f_1998 > f_1997 THEN 'CRECIÓ'
        WHEN f_1998 < f_1997 THEN 'DECRECIÓ'
        ELSE 'SIN CAMBIO'
    END AS tendencia

FROM con_total
ORDER BY es_total, categoria;
```

![Resultado de la pregunta 20](20.PNG)
