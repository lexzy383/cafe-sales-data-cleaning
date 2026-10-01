# cafe-sales-data-cleaning
-- ====================================================================
-- PROJECT: Cafe Sales Data Cleaning & Standardization
-- ARCHITECTURE: MySQL / SQL Server
-- DESCRIPTION: Cleaning raw, dirty transaction data by removing duplicates,
--              imputing missing values, and formatting data types.
-- ====================================================================

-- 1. INITIAL DATA INSPECTION
SELECT * FROM dirty_cafe_sales;

-- Create a staging table to preserve the original raw data
CREATE TABLE dirty_cafe_sales_1 LIKE dirty_cafe_sales;
INSERT INTO dirty_cafe_sales_1 SELECT * FROM dirty_cafe_sales;

SELECT * FROM dirty_cafe_sales_1;


-- 2. DUPLICATE DETECTION 
-- Identifying duplicate transactions using standard window functions
SELECT *, 
       ROW_NUMBER() OVER(
           PARTITION BY `transaction ID`, item, quantity, `price per unit`, 
                        `total spent`, `payment method`, location, `transaction date`
       ) AS row_num 
FROM dirty_cafe_sales_1;

-- Isolate the duplicates using a Common Table Expression (CTE)
WITH duplicate_cte AS (
    SELECT *, 
           ROW_NUMBER() OVER(
               PARTITION BY `transaction ID`, item, quantity, `price per unit`, 
                            `total spent`, `payment method`, location, `transaction date`
           ) AS row_num 
    FROM dirty_cafe_sales_1
) 
SELECT * FROM duplicate_cte WHERE row_num > 1;


-- 3. DATA STANDARDIZATION & IMPUTATION

-- Step A: Impute missing/corrupted item names based on unit pricing mapping
SELECT `Price Per Unit`, COUNT(*) AS transaction_count 
FROM dirty_cafe_sales_1 
WHERE item = 'unknown' 
GROUP BY `price per unit`;

SELECT DISTINCT item, `price per unit` 
FROM dirty_cafe_sales_1 
WHERE item != 'unknown' AND `price per unit` IN (3, 1, 5, 4, 1.5, 2) 
ORDER BY `price per unit`;

UPDATE dirty_cafe_sales_1 
SET item = CASE 
    WHEN `price per unit` = 1.5 THEN 'tea' 
    WHEN `price per unit` = 2   THEN 'coffee' 
    WHEN `price per unit` = 5   THEN 'salad' 
    WHEN `price per unit` = 3   THEN 'cake or juice' 
    WHEN `price per unit` = 4   THEN 'sandwich or smoothie' 
    ELSE 'unknown item' 
END 
WHERE item = 'unknown' OR item = 'error' OR item IS NULL OR item = '';

-- Step B: Correct corrupt numerical data in the Total Spent column
SELECT DISTINCT `Quantity`, `price per unit`, COUNT(*) AS transaction_count 
FROM dirty_cafe_sales_1 
WHERE `Total Spent` = 'unknown' 
GROUP BY `quantity`, `price per unit`;

UPDATE dirty_cafe_sales_1 
SET `total spent` = `quantity` * `price per unit` 
WHERE `total spent` REGEXP '[a-zA-Z]' OR `total spent` IS NULL OR `total spent` = 0;

-- Step C: Standardize and clean categorical fields (Payment Method & Location)
UPDATE dirty_cafe_sales_1 
SET `payment method` = 'unknown' 
WHERE `payment method` IN ('unknown', 'error', '') OR `payment method` IS NULL OR TRIM(`payment method`) = '';

SELECT `payment method`, COUNT(*) AS transaction_count FROM dirty_cafe_sales_1 GROUP BY `payment method`;

UPDATE dirty_cafe_sales_1 
SET location = 'unknown' 
WHERE location IN ('unknown', 'error', '') OR location IS NULL OR TRIM(location) = '';

SELECT location, COUNT(*) AS transaction_count FROM dirty_cafe_sales_1 GROUP BY location;


-- 4. DATE FORMATTING & TYPE CASTING
SELECT `transaction date` FROM dirty_cafe_sales_1;

-- Clear corrupted text values in the date field
UPDATE dirty_cafe_sales_1 
SET `transaction date` = NULL 
WHERE `transaction date` IN ('error', 'unknown', '') OR TRIM(`transaction date`) = '';

-- Convert strings to standard SQL Date format
UPDATE dirty_cafe_sales_1 
SET `transaction date` = STR_TO_DATE(`transaction date`, '%d-%m-%Y') 
WHERE `transaction date` IS NOT NULL;

-- Permanently modify the column structure to DATE type
ALTER TABLE dirty_cafe_sales_1 MODIFY COLUMN `Transaction Date` DATE;

-- Final Verification
SELECT * FROM dirty_cafe_sales_1;
