# sql_coding
Расчеты, анализ, код на языке sql

---
Для расчета использовать общедоступные данные [bookings](https://postgrespro.ru/education/demodb) "Авиаперевозки"
```
-- Рассчитывал LTV по формуле: Средний чек х частота покупок х время жизни
WITH passenger_lifetime AS (
SELECT
	t.passenger_id,
	t.passenger_name,
	b.total_amount,
	AVG(b.total_amount) AS avg_revenue,
	COUNT(DISTINCT b.book_ref) AS frequency_purchase,
-- взял время жизни клиента с момента бронирования до вылета, что гарантирует выполенине сделки
	EXTRACT(EPOCH FROM (MAX(f.scheduled_departure) - MIN(b.book_date))) / 86400 AS lifetime 
FROM bookings.tickets t
LEFT JOIN bookings.bookings b ON t.book_ref = b.book_ref
LEFT JOIN bookings.ticket_flights tf ON t.ticket_no = tf.ticket_no
LEFT JOIN bookings.flights f ON tf.flight_id = f.flight_id
WHERE f.status <> 'Cancelled'
GROUP BY
	t.passenger_id,
	t.passenger_name,
	b.total_amount
)
SELECT
	passenger_id,
	passenger_name,
	total_amount,
	ROUND(avg_revenue * frequency_purchase * lifetime) AS ltv
FROM passenger_lifetime
ORDER BY ltv DESC
```
---
