# Business Question 1 — Top 10 Revenue-Generating Products

### 🎯 Objective
Identify the Top 10 products generating the highest revenue.

### 💼 Why does this matter?
High-revenue products are key revenue drivers. Identifying them helps Flipkart optimize inventory planning, prioritize marketing efforts, and ensure product availability.

### 🧠 Approach
- Join `products` and `order_items` using `product_id`
- Calculate revenue as `quantity × price`
- Aggregate revenue using `SUM()`
- Group by product
- Sort by revenue in descending order
- Return the Top 10 products

### 💻 SQL

```sql
SELECT
    p.product_id,
    p.category,
    p.product_type,
    SUM(oi.quantity * oi.price) AS revenue
FROM flipkart.order_items oi
JOIN flipkart.products p
ON oi.product_id = p.product_id
GROUP BY
    p.product_id,
    p.category,
    p.product_type
ORDER BY revenue DESC
LIMIT 10;
```

### 💡 Business Recommendation

Prioritize high-revenue products by maintaining sufficient inventory, featuring them in promotional campaigns, and monitoring demand to minimize stock-outs.

---


# Business Question 2 — Top 10 Best-Selling Products by Quantity Sold

### 🎯 Objective

Identify the Top 10 best-selling products based on the total quantity sold.

### 💼 Why does this matter?

Best-selling products indicate customer demand and purchasing behavior. Identifying these products helps Flipkart:

- Optimize inventory levels
- Prevent stock-outs
- Improve demand forecasting
- Prioritize marketing campaigns
- Negotiate better pricing with suppliers

---

## ✅ Solution 1 — Production Approach (Recommended)

### 🧠 Approach

The business only asked for the **Top 10 products**.

Since rankings are not required, the simplest and most efficient solution is to:

- Join `products` and `order_items`
- Calculate the total quantity sold using `SUM(quantity)`
- Group by product
- Sort by quantity sold
- Return the first 10 rows using `LIMIT`

### 💻 SQL

```sql
SELECT
    p.product_id,
    p.product_type,
    SUM(oi.quantity) AS total_quantity
FROM flipkart.order_items oi
JOIN flipkart.products p
ON oi.product_id = p.product_id
GROUP BY
    p.product_id,
    p.product_type
ORDER BY total_quantity DESC
LIMIT 10;
```

### 💡 Why this is the preferred solution

This query is:

- Easier to read
- Faster to execute
- Simpler to maintain
- Exactly matches the business requirement

Whenever the business simply asks for the "Top N" records, this is the preferred approach.

---

## ✅ Solution 2 — Using Window Functions (Learning Approach)

### 🧠 Approach

Instead of limiting the results directly, assign a rank to every product based on the quantity sold.

This approach is useful when the business wants:

- Product rankings
- Leaderboards
- To include ties
- Reporting dashboards

### 💻 SQL

```sql
WITH best_selling AS (
    SELECT
        p.product_id,
        p.product_type,
        SUM(oi.quantity) AS total_quantity
    FROM flipkart.order_items oi
    JOIN flipkart.products p
        ON oi.product_id = p.product_id
    GROUP BY
        p.product_id,
        p.product_type
),

ranked_products AS (
    SELECT
        *,
        DENSE_RANK() OVER (ORDER BY total_quantity DESC) AS product_rank
    FROM best_selling
)

SELECT *
FROM ranked_products
WHERE product_rank <= 10;
```

### 💡 Why use DENSE_RANK()?

Unlike `LIMIT`, `DENSE_RANK()` handles ties fairly.

Example:

| Product | Quantity | Rank |
|---------|---------:|-----:|
| Laptop | 120 | 1 |
| Mobile | 115 | 2 |
| Speaker | 100 | 3 |
| Sandals | 95 | 4 |
| Shoes | 95 | 4 |
| Watch | 95 | 4 |

If multiple products have the same quantity sold, they receive the same rank.

As a result, filtering with:

```sql
WHERE product_rank <= 10
```

may return **more than 10 rows**, ensuring that tied products are not excluded unfairly.

---

## 🏆 Which approach should you use?

For this business question, **Solution 1 is the better choice** because the stakeholder only requested the Top 10 products.

Use **Solution 2** when rankings or tied positions are important, such as leaderboards, executive reports, or performance analysis.

---

### 💡 Business Recommendation

Ensure the highest-selling products are consistently available in inventory, prioritize them in promotional campaigns, and monitor demand trends to minimize stock-outs during peak sales periods.

---

# Business Question 3 — Product Category Generating the Highest Revenue

### 🎯 Objective

Identify the product category that generates the highest total revenue.

### 💼 Why does this matter?

Understanding which product category contributes the most revenue helps Flipkart:

- Allocate marketing budgets effectively
- Prioritize inventory investments
- Focus on high-performing categories
- Improve category-level business strategy

### 🧠 Approach

- Join `products` and `order_items` using `product_id`
- Calculate revenue as `quantity × price`
- Aggregate revenue by category
- Sort categories by total revenue
- Return the highest revenue-generating category

### 💻 SQL

```sql
SELECT
    p.category,
    SUM(oi.quantity * oi.price) AS total_revenue
FROM flipkart.order_items oi
JOIN flipkart.products p
ON oi.product_id = p.product_id
GROUP BY
    p.category
ORDER BY total_revenue DESC
LIMIT 1;
```

### 💡 Business Recommendation

Prioritize the highest-performing category by ensuring adequate inventory, increasing marketing investment, and expanding the product assortment to maximize revenue growth.

---

# Business Question 4 — Highest Revenue-Generating Product in Each Category

### 🎯 Objective

Identify the highest revenue-generating product within each product category.

### 💼 Why does this matter?

Identifying the top-performing product in each category helps Flipkart:

- Identify category leaders
- Prioritize inventory for flagship products
- Allocate marketing budgets effectively
- Benchmark other products within the same category

### 🧠 Approach

- Join `products` and `order_items` using `product_id`
- Calculate product revenue as `quantity × price`
- Aggregate revenue for each product
- Use `ROW_NUMBER()` with `PARTITION BY category` to rank products within each category
- Return only the highest revenue-generating product from each category

### 💻 SQL

```sql
WITH product_revenue AS (
    SELECT
        p.category,
        p.product_id,
        p.product_type,
        SUM(o.quantity * o.price) AS revenue
    FROM `flipkart.products` p
    JOIN `flipkart.order_items` o
        ON p.product_id = o.product_id
    GROUP BY
        p.category,
        p.product_id,
        p.product_type
),

ranked_products AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY category
            ORDER BY revenue DESC
        ) AS rn
    FROM product_revenue
)

SELECT
    category,
    product_id,
    product_type,
    revenue
FROM ranked_products
WHERE rn = 1;
```

### 💡 Business Recommendation

Use the highest-performing product in each category as a flagship product for promotions, ensure consistent inventory availability, and analyze the factors contributing to its success to improve the performance of other products in the same category.

---

# Business Question 5 — Products with the Highest Return Rate

### 🎯 Objective

Identify the products with the highest return rate based on actual transaction data.

### 💼 Why does this matter?

Products with high return rates can negatively impact profitability, customer satisfaction, and operational costs. Identifying them helps Flipkart:

- Detect quality issues
- Improve supplier performance
- Reduce refund costs
- Improve customer experience

### 🧠 Approach

- Join `products` with `order_items` using `product_id`
- Left join `return_status` using `order_id`
- Count total orders for each product
- Count returned orders for each product
- Calculate the return rate as:

  **(Returned Orders / Total Orders) × 100**

- Rank products by return rate

### 💻 SQL

```sql
SELECT
    p.product_id,
    p.product_type,
    COUNT(DISTINCT oi.order_id) AS total_orders,
    COUNT(DISTINCT rs.order_id) AS returned_orders,
    ROUND(
        COUNT(DISTINCT rs.order_id) * 100.0 /
        COUNT(DISTINCT oi.order_id),
        2
    ) AS return_rate
FROM flipkart.products p
JOIN flipkart.order_items oi
    ON p.product_id = oi.product_id
LEFT JOIN flipkart.return_status rs
    ON oi.order_id = rs.order_id
GROUP BY
    p.product_id,
    p.product_type
ORDER BY return_rate DESC;
```

### 💡 Business Recommendation

Investigate products with the highest return rates to identify root causes such as product quality, inaccurate descriptions, supplier issues, or shipping damage. Reducing return rates can improve customer satisfaction while lowering logistics and refund costs.

---

# Business Question 6 — High Revenue Products with Low Customer Ratings

### 🎯 Objective

Identify products that generate high revenue despite having below-average customer ratings.

### 💼 Why does this matter?

Products with strong sales but poor ratings may indicate underlying quality or customer experience issues. Identifying them helps Flipkart:

- Detect products at risk of losing future sales
- Improve product quality
- Reduce negative customer reviews
- Improve long-term customer satisfaction

### 🧠 Approach

- Calculate the average rating across all products
- Join `products` and `order_items` using `product_id`
- Calculate revenue as `quantity × price`
- Filter products whose rating is below the overall average
- Sort products by revenue in descending order

### 💻 SQL

```sql
WITH avg_rating AS (
    SELECT AVG(average_rating) AS avg_rating
    FROM flipkart.products
)

SELECT
    p.product_id,
    p.product_type,
    SUM(o.quantity * o.price) AS revenue,
    p.average_rating
FROM flipkart.products p
JOIN flipkart.order_items o
    ON p.product_id = o.product_id
WHERE p.average_rating < (
    SELECT avg_rating
    FROM avg_rating
)
GROUP BY
    p.product_id,
    p.product_type,
    p.average_rating
ORDER BY revenue DESC;
```

### 💡 Business Recommendation

Investigate products generating high revenue despite poor customer ratings. Improve product quality, packaging, or descriptions before customer dissatisfaction begins to impact future sales and brand reputation.

---

# Business Question 7 — Product Categories with the Highest Average Customer Ratings

### 🎯 Objective

Identify the product categories with the highest average customer ratings.

### 💼 Why does this matter?

Highly rated product categories indicate strong customer satisfaction and product quality. Identifying these categories helps Flipkart:

- Invest more in high-performing categories
- Identify best-performing suppliers
- Improve lower-rated categories
- Enhance customer trust and retention

### 🧠 Approach

- Calculate the average customer rating for each category
- Group products by category
- Sort categories by average rating in descending order

### 💻 SQL

```sql
SELECT
    category,
    ROUND(AVG(average_rating), 2) AS avg_customer_rating
FROM flipkart.products
GROUP BY category
ORDER BY avg_customer_rating DESC;
```

### 💡 Business Recommendation

Focus marketing and inventory investments on highly rated categories while analyzing lower-rated categories to identify opportunities for product quality improvements, better supplier management, or enhanced customer experience.

---

# Business Question 8 — Products Priced Above the Average Product Price

### 🎯 Objective

Identify products priced above the average product price.

### 💼 Why does this matter?

Premium-priced products contribute to revenue and profit but may require different pricing and marketing strategies. Identifying these products helps Flipkart:

- Analyze premium product performance
- Evaluate pricing strategies
- Design targeted marketing campaigns
- Monitor premium inventory

### 🧠 Approach

- Calculate the average product price using a subquery
- Compare each product's price with the overall average
- Return products priced above the average

### 💻 SQL

```sql
SELECT
    product_id,
    product_type,
    price
FROM flipkart.products
WHERE price > (
    SELECT AVG(price)
    FROM flipkart.products
);
```

### 💡 Business Recommendation

Analyze whether premium-priced products justify their pricing through strong customer ratings and sales performance. High-priced products with poor ratings may require pricing or quality improvements.

---

# Business Question 9 — Products That Have Never Been Ordered

### 🎯 Objective

Identify products that have never been ordered by customers.

### 💼 Why does this matter?

Products that have never been purchased may indicate poor demand, ineffective pricing, lack of visibility, or inventory issues. Identifying these products helps Flipkart:

- Reduce dead inventory
- Improve product recommendations
- Optimize inventory costs
- Re-evaluate pricing and marketing strategies

### 🧠 Approach

- Start with the `products` table to include all products
- Left join `order_items` using `product_id`
- Identify products with no matching order records
- Filter using `WHERE order_id IS NULL`

### 💻 SQL

```sql
SELECT
    p.product_id,
    p.product_type,
    p.category
FROM flipkart.products p
LEFT JOIN flipkart.order_items oi
ON p.product_id = oi.product_id
WHERE oi.order_id IS NULL;
```

### 💡 Business Recommendation

Review products that have never been ordered to determine whether they should be promoted, repriced, bundled with other products, or discontinued to reduce inventory holding costs.

---


