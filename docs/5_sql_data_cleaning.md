# SQL data cleaning

## Introduction

- Clean and re-structure messy data.
- Convert columns to different data types.
- Tricks for manipulating NULLs.
- Functions covered
    - `LEFT` - extracts the number of characters from a string starting from the left
    - `RIGHT` - extracts the number of characters from a string starting from the right
    - `SUBSTR` - extracts the substring from a string (starting at any position)
    - `POSITION` - returns the position of the first occurence of a substring in a string
    - `STRPOS` - Returns the position of substring within a string
    - `CONCAT` - adds two or more expressions together
    - `CAST` - covers a value of any type into a specific, different data type
    - `COALESCE` - returns the first non-null value in a list

## Data Cleaning Strategy

1. What data do you need?: Review what data you need to run an analysis and solve the problem at hand.
2. What data do you have?: Take stock of not only the information you have in your dataset today but what data types those fields are. Do these align with your data needs?
3. How will you clean your data?: Build a game plan of how you’ll convert the data you currently have to the data you need. What types of actions and data cleaning techniques will you have to apply? Do you have the skills you need to go from the current to future state?
4. How will you analyze your data?: Now, it’s game time! How do you run an effective analysis? Build an approach for analysis, as well. And visualize your plan to solve the problem. Finally, remember to question “so what?” at the end of your results, which will help drive recommendations for your organization.

```sql
SELECT LEFT(primary_poc, STRPOS(primary_poc, ' ') -1 ) first_name, 
       RIGHT(primary_poc, LENGTH(primary_poc) - STRPOS(primary_poc, ' ')) last_name
FROM accounts;
```

## Commonly used function

### LEFT

```sql
LEFT(string, number_of_character) AS column_name
```

___Note___ number_of_characters is inclusive i.e. LEFT('ABC', 2) => 'AB'

### RIGHT

```sql
RIGHT(string, number_of_character) AS column_name
```

___Note___ number_of_characters is inclusive i.e. RIGHT('ABC', 2) => 'BC'


### SUBSTR

```sql
SUBSTR(string, start, length)
```

- string: the string you are parsing
- start: the start position of the extraction
- length: number of characters to extract

```sql
SUBSTR(col, 11, 1) AS gender
```

### STRING_SPLIT

```sql
WITH table AS(
SELECT  student_information,
        value,
        ROW _NUMBER() OVER(PARTITION BY student_information ORDER BY (SELECT NULL)) AS row_number
FROM    student_db
        CROSS APPLY STRING_SPLIT(student_information, ',') AS back_values
)
SELECT  student_information,
        [1] AS STUDENT_ID,
        [2] AS GENDER,
        [3] AS CITY,
        [4] AS GPA,
        [5] AS SALARY
FROM    table
PIVOT(
        MAX(VALUE)
        FOR row_number IN([1],[2],[3],[4],[5])
) AS PVT)
```

### CONCAT

```sql
CONCAT(string1, string2, string3)
```

```sql
CONCAT(month, '-', day, '-', year) AS date
```

### CAST

```sql
CAST(expression as datatype)
```

```sql
CAST(salary AS int)
```

```sql
SELECT date orig_date, (SUBSTR(date, 7, 4) || '-' || LEFT(date, 2) || '-' || SUBSTR(date, 4, 2))::DATE new_date -- some other way you can cast 
FROM sf_crime_data;
```

### POSITION

```sql
POSITION(substring IN string)
```

```sql
POSITION("$" IN student_information) as salary_starting_position
```

### STRPOS

```sql
STRPOS(string, substring)
```

```sql
SELECT LEFT(name, STRPOS(name, ' ') -1 ) first_name, 
       RIGHT(name, LENGTH(name) - STRPOS(name, ' ')) last_name
FROM sales_reps;
```

### COALESCE

```sql
COALESCE(val1, val2, val3)
```

```sql
COALESCE(hourly_wage*40*52, salary, commission*sales) AS annual_income
```