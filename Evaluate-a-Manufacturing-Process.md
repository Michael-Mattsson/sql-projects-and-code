# Manufacturing Process Evaluation Using Statistical Process Control (SPC)

## Project Overview

This project focuses on improving manufacturing quality by applying **Statistical Process Control (SPC)** techniques.

Using SQL and historical production data, we analyze product measurements to calculate the:

- **Upper Control Limit (UCL)**
- **Lower Control Limit (LCL)**

These control limits define the expected operating range of the manufacturing process. Products falling outside this range are flagged as potential quality issues requiring investigation.

This data-driven approach helps:

- Maintain consistent product quality
- Detect process variation
- Reduce defects
- Improve manufacturing efficiency

---

## Statistical Method

The control limits are calculated using:

$$
UCL = avg\_height + 3 \times \frac{stddev\_height}{\sqrt{5}}
$$

$$
LCL = avg\_height - 3 \times \frac{stddev\_height}{\sqrt{5}}
$$

Where:

- **UCL** = Maximum acceptable product height
- **LCL** = Minimum acceptable product height
- **avg_height** = Average product height
- **stddev_height** = Standard deviation of product height

Ideally, all products should fall between the UCL and LCL.

---

# Data Source

The analysis uses the `manufacturing_parts` table.

| Column | Description |
|---|---|
| `item_no` | Unique item identifier |
| `length` | Length of manufactured part |
| `width` | Width of manufactured part |
| `height` | Height of manufactured part |
| `operator` | Machine/operator responsible for production |

---

# Solution 1: Common Table Expression (CTE)

The first approach calculates rolling averages and standard deviations using SQL window functions.

```sql
WITH f AS (

SELECT 
    operator,
    ROW_NUMBER() OVER (
        PARTITION BY operator 
        ORDER BY item_no
    ) AS row_number,

    height,

    AVG(height) OVER(
        PARTITION BY operator
    ) AS avg_height,

    STDDEV(height) OVER(
        PARTITION BY operator
    ) AS stddev_height,

    AVG(height) OVER(
        PARTITION BY operator 
        ORDER BY item_no 
        ROWS BETWEEN 4 PRECEDING AND CURRENT ROW
    ) 
    + 
    3 * (
        STDDEV(height) OVER(
            PARTITION BY operator 
            ORDER BY item_no 
            ROWS BETWEEN 4 PRECEDING AND CURRENT ROW
        ) / SQRT(5)
    ) AS ucl,

    AVG(height) OVER(
        PARTITION BY operator 
        ORDER BY item_no 
        ROWS BETWEEN 4 PRECEDING AND CURRENT ROW
    )
    -
    3 * (
        STDDEV(height) OVER(
            PARTITION BY operator 
            ORDER BY item_no 
            ROWS BETWEEN 4 PRECEDING AND CURRENT ROW
        ) / SQRT(5)
    ) AS lcl,

    CASE
        WHEN height > ucl
        OR height < lcl
        THEN TRUE
        ELSE FALSE
    END AS alert

FROM manufacturing_parts

)

SELECT *
FROM f
WHERE row_number >= 5
ORDER BY row_number;
```

---

# Solution 2: Cleaner Approach Using Subqueries

A more compact solution separates the calculations into logical steps:

1. Calculate rolling averages and standard deviation
2. Calculate control limits
3. Flag out-of-control measurements

```sql
-- Flag whether product height is within control limits

SELECT
    b.*,

    CASE
        WHEN height NOT BETWEEN lcl AND ucl
        THEN TRUE
        ELSE FALSE
    END AS alert

FROM (

    SELECT
        a.*,

        avg_height 
        + 3 * stddev_height / SQRT(5) AS ucl,

        avg_height 
        - 3 * stddev_height / SQRT(5) AS lcl

    FROM (

        SELECT

            operator,

            ROW_NUMBER() OVER w AS row_number,

            height,

            AVG(height) OVER w AS avg_height,

            STDDEV(height) OVER w AS stddev_height

        FROM manufacturing_parts

        WINDOW w AS (

            PARTITION BY operator

            ORDER BY item_no

            ROWS BETWEEN 4 PRECEDING AND CURRENT ROW

        )

    ) AS a

    WHERE row_number >= 5

) AS b;
```

---

# Key Learnings

This project demonstrates:

- SQL window functions
- Rolling calculations
- Common Table Expressions (CTEs)
- Statistical process control
- Quality monitoring using data analysis

The final output identifies when a manufacturing process may be producing parts outside acceptable limits, allowing operators to investigate potential issues before defects increase.
