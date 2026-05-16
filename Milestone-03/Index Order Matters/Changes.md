# Document your index fixes here

## Step 1-3: Analysis of the Original Index
- **Original index:** `CREATE INDEX idx_salary_department ON employees(salary, department);`
- **Queries tested:** 
  ```sql
  SELECT * FROM employees WHERE department = 'Sales' AND salary > 50000;
  ```
- **Issue observed:** The PostgreSQL planner performs a `Seq Scan` (or an inefficient `Index Scan` processing far too many rows) instead of optimally using the index. 
- **Why it was ineffective:** Because the index starts with `salary` (a range condition column) and then `department` (an equality condition column), it violates the left-most prefix rule best practices. The index tree is sorted by `salary` first. To find `department = 'Sales'`, the database cannot perform a quick lookup; instead, it has to scan through all index entries where `salary > 50000` and then filter out the rows where `department` isn't 'Sales'.

## Step 4-5: Explanation of the Optimization
- **Fixed index:** `CREATE INDEX idx_department_salary ON employees(department, salary);`
- **Performance improvement:** The query uses an efficient `Index Scan` (or `Bitmap Index Scan`). 
- **The Left-Most Prefix Rule:** This rule dictates that a query can only effectively use a composite index if the query conditions include the first (left-most) columns of the index. For optimal filtering, columns used with equality operators (`=`) should appear first in the index, followed by columns used with range operators (`<`, `>`, `BETWEEN`, etc.). By swapping the order to `(department, salary)`, the database can instantly jump to the `department = 'Sales'` section of the B-tree, and then traverse sequentially over the `salary > 50000` section, vastly reducing the number of blocks read from disk.