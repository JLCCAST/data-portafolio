## Clientes por fecha
**Problema:** listar clientes ordenado por fecha.

```sql
select p.fecha, c.nombre, c.ciudad, p.estado from 
diag.pedidos p inner join 
diag.clientes c ON p.cliente_id = c.cliente_id
order by p.fecha 
```

**Cuidado:** Solo si hay mas campos de fecha

## Registros sin coincidencia
**Problema:** clientes que nunca hicieron un pedido.

```sql
select c.nombre from
diag.clientes c  left join 
diag.pedidos p on p.cliente_id = c.cliente_id 
where p.cliente_id is null
```

**Cuidado:** a la izquierda va la tabla de la que quiero todas las filas.
Se usa IS NULL, nunca = NULL.

## Registro de ingresos
**Problema:** Obtener ingresos por categeoria ordenado de mayor a menor

```sql
SELECT 
    SUM(dp.cantidad * dp.precio_unitario) AS ingresos,
    p2.categoria
FROM diag.detalle_pedido dp
JOIN diag.productos p2 ON p2.producto_id = dp.producto_id
JOIN diag.pedidos p ON p.pedido_id = dp.pedido_id
WHERE p.estado <> 'Cancelado'
GROUP BY p2.categoria
ORDER BY ingresos DESC;   
```

**Cuidado:** Se usa <> para diferenciar

## Cantidad de pedidos
**Problema:** Listar la cantidad de clientes con 4 a mas pedidos

```sql
select c.nombre, count(c.nombre) as numero_pedidos
from diag.clientes c 
join diag.pedidos p on c.cliente_id = p.cliente_id 
group by c.nombre
having COUNT(c.nombre) > 4
```

**Cuidado:** No se puede usar el "numero_pedidos" en lugar de count(*) en el having

## Clasificación
**Problema:** Clasificar productos deacuerdo a su precio y asignarle una categoria

```sql
select 
	case 
		when p.precio < 50 then 'Economico'
		when p.precio >= 50 and p.precio <= 200 then 'Medio'
		else 'Premium'
	end as Segmento,
	count (*)
from diag.productos p
group by segmento 
```

**Cuidado:** El CASE va dentro del SELECT

## Sacar promedio
**Problema:** Saber cuales son los producots con el precio superior al promedio

```sql
SELECT p.nombre, p.precio
FROM diag.productos p
WHERE p.precio > (SELECT AVG(precio) FROM diag.productos);   
```

**Cuidado:** La subconsulta no se hace en el select, si no en el where

## Productos clasificados
**Problema:** clientes que cumplen algo al menos una vez

```sql
SELECT c.cliente_id, c.nombre
FROM diag.clientes c
WHERE EXISTS (
  SELECT 1
  FROM diag.pedidos p
  JOIN diag.detalle_pedido dp ON dp.pedido_id = p.pedido_id
  JOIN diag.productos p2 ON p2.producto_id = dp.producto_id
  WHERE p.cliente_id = c.cliente_id
    AND p.estado <> 'Cancelado'
    AND p2.categoria = 'Electrónica'
);
```

**Cuidado:** EXISTS no duplica clientes y no necesita GROUP BY

