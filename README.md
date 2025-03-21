# SQL-Layoffs-Data-Cleaning

Overview
This project focuses on cleaning and preparing the layoffs_staging2 dataset by removing duplicates, standardizing data, handling NULL values, and dropping unnecessary rows or columns. The goal is to ensure data accuracy, consistency, and usability for further analysis.

Project Objectives
•	Remove duplicate records to ensure data integrity.
•	Standardize data formats for consistency.
•	Handle NULL values to improve dataset completeness.
•	Remove unnecessary rows and columns for better performance and readability

Dataset Used
•	Layoffs Dataset 
•	Source:  https://github.com/taiyk53/SQL-Layoffs-Data-Cleaning/blob/main/layoffs.csv
•	Description: The dataset contains information about company layoffs, including details like company name, industry, location, total layoffs, percentage laid off, funding raised, and dates.

**Data Cleaning Steps**

1. Removing Duplicates
•	Created a staging table (layoffs_staging2) to store cleaned data.
•	Identified duplicates using ROW_NUMBER() with PARTITION BY on relevant columns.
•	Deleted duplicate entries to maintain data integrity.

**SQL Syntax for Removing Duplicates**

-- Identify and Insert Data -- 
SELECT * FROM layoffs_staging;
INSERT INTO layoffs_staging SELECT * FROM layoffs;

-- Identify Duplicates -- 
SELECT *, ROW_NUMBER() OVER(
 PARTITION BY company, industry, total_laid_off, percentage_laid_off, date) AS row_num
FROM layoffs_staging;

-- Remove Duplicates Using CTE -- 
WITH duplicate_cte AS (
  SELECT *, ROW_NUMBER() OVER(
    PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, date, stage, country, funds_raised_millions) AS row_num
  FROM layoffs_staging
)
DELETE FROM duplicate_cte WHERE row_num > 1;

2. Standardizing Data
•	Trimmed unnecessary spaces from company names.
•	Standardized industry names (e.g., ensuring consistency in naming like "Crypto").
•	Removed trailing dots from country values.
•	Converted date from text format to proper DATE format using STR_TO_DATE().

 **SQL Syntax for Standardizing Data**
 
-- Trim Whitespaces -- 
UPDATE layoffs_staging2 SET company = TRIM(company);

-- Standardize Industry Names -- 
UPDATE layoffs_staging2 SET industry = 'Crypto' WHERE industry LIKE 'Crypto%';

-- Clean Country Names -- 
UPDATE layoffs_staging2 SET country = TRIM(TRAILING '.' FROM country) WHERE country LIKE 'United States%';

-- Convert Date Format -- 
UPDATE layoffs_staging2 SET date = STR_TO_DATE(date, '%m/%d/%Y');
ALTER TABLE layoffs_staging2 MODIFY COLUMN date DATE;

3. Handling NULL Values
•	Identified records where total_laid_off and percentage_laid_off were NULL.
•	Replaced blank industry values with NULL.
•	Used self-joins to fill missing industry values based on other records of the same company.

**SQL Syntax for Handling NULL Values**

-- Identify NULL Values -- 
SELECT * FROM layoffs_staging2 WHERE total_laid_off IS NULL AND percentage_laid_off IS NULL;

-- Update NULL Industry Values -- 
UPDATE layoffs_staging2 SET industry = NULL WHERE industry = '';

-- Fill Missing Industry Data Based on Company -- 
UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2 ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL AND t2.industry IS NOT NULL;

4. Removing Unnecessary Rows and Columns
•	Removed rows where both total_laid_off and percentage_laid_off were NULL.
•	Dropped the row_num column as it was only used for duplicate identification.

**SQL Syntax for Removing Unnecessary Rows and Columns**

-- Delete NULL Rows -- 
DELETE FROM layoffs_staging2 WHERE total_laid_off IS NULL AND percentage_laid_off IS NULL;

-- Drop Unnecessary Columns -- 
ALTER TABLE layoffs_staging2 DROP COLUMN row_num;

Conclusion
This data cleaning process provides that the layoffs dataset is:
Accurate (No duplicate records)
Consistent (Standardized formats)
Complete (Missing values handled)
Optimized (Unnecessary data removed)

🛠 Technologies Used
SQL (MySQL)
Data Cleaning Techniques: Deduplication, Standardization, NULL Handling, Column/Row Removal


