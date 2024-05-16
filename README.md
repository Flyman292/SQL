# SQL

WITH PreviousActiveOrders AS (
    SELECT
        a.user_id,
        a.date_start AS current_date_start,
        a.premium AS current_premium,
        SUM(b.premium) OVER (PARTITION BY a.user_id ORDER BY b.date_start) AS total_previous_order
    FROM
        INSURANCE_ORDERS a
    LEFT JOIN
        INSURANCE_ORDERS b
    ON
        a.user_id = b.user_id
        AND b.date_start < a.date_start
        AND b.date_end >= a.date_start
    GROUP BY
        a.user_id, a.date_start, a.premium
        )
      
	SELECT 
    o.date_start,
    o.date_end,
    o.user_id,
    o.order_rank,
    o.premium,
    pa.PreviousActiveOrders + o.premium AS active_premium
 	CASE
    	WHEN active_premium >=30000 THEN 'TRUE' ELSE 'FALSE' END AS flag
    FROM 
    	INSURANCE_ORDERS o
    LEFT JOIN 
    	PreviousActiveOrders pa
    ON 
    o.user_id = pa.user_id
    ORDER BY
    o.user_id, o.order_rank;
    
    
    
    
    WITH UserOrders AS (
    SELECT
        user_id,
        date_start,
        premium,
        SUM(premium) OVER (PARTITION BY user_id ORDER BY date_start) AS accumulated_premium
    FROM
        INSURANCE_ORDERS
    WHERE
        user_id = '9027429831' -- Filter orders for the specific user
)
SELECT
    user_id,
    date_start,
    premium,
    accumulated_premium AS active_premium,
    CASE WHEN accumulated_premium >= 300000 THEN 'TRUE' ELSE 'FALSE' END AS flag
FROM
    UserOrders
WHERE
    date_start = '2023-09-01'; -- Retrieve the 8th order by filtering on the date_start




   WITH UserOrders AS (
    SELECT
        user_id,
        date_start,
        premium,
        SUM(premium) OVER (PARTITION BY user_id ORDER BY date_start ROWS BETWEEN UNBOUNDED PRECEDING AND 1 PRECEDING) AS accumulated_premium
    FROM
        INSURANCE_ORDERS
    WHERE
        user_id = '9027429831' -- Filter orders for the specific user
        AND date_end >= '2023-09-01' -- Include only orders active by 1st Sept 2023
)
SELECT
    user_id,
    date_start,
    premium,
    accumulated_premium + premium AS active_premium,
    CASE WHEN (accumulated_premium + premium) >= 300000 THEN 'TRUE' ELSE 'FALSE' END AS flag
FROM
    UserOrders
WHERE
    order_rank = 9; -- Retrieve the 9th order



    
