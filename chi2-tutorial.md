# From Nested Loops to Matrix Math: A Chi-Square Walkthrough

## Who this tutorial is for

This tutorial is for Python learners who can already write a `for` loop and define a function, but have not yet leaned on NumPy for everyday matrix work.

You should already know:

- How to define and call Python functions
- How to write `for` loops and use `range`
- How to index into lists and lists of lists
- How to run cells in a Jupyter notebook

You do not need to know (if you do know these call it a bonus):

- NumPy or pandas
- Probability or hypothesis testing (the one paragraph below the scenario is enough)
- Anything specific about chi-square

## How to work through this tutorial

Each step in this tutorial is a single notebook cell, labeled either markdown (for explanatory text) or code (for Python you can run). Work through the steps below, putting each block of code or text in its own notebook cell. Run the cells in order.

Copying the code is encouraged. Reading code, copying it, and then poking at it is how a programming language sinks in. Note that where "poking at it" means making educated thoughtful chances to the code on your own before re-running the code.

## What you will build

You are going to compute a chi-square test of independence from scratch, twice:

1. First with nested `for` loops. For loops are a manner of programming that many newer programmers would choose.
2. Then with NumPy matrix operations. Matrix operations are a more advanced and professional approach.

Both approaches work. The point of putting them side by side is to make you fluent enough to choose between them on purpose, not by habit. `for` loops are not an enemy (they are not inherently 'bad' or 'flawed'). You will use `for` for the rest of your career. They are simply not always the right the most efficient choice.

Along the way you will produce four named results:

- the expected matrix
- the normalized error matrix
- the sum of the normalized errors (this number is the chi-square statistic)
- the p-value for that sum

"Named results" can be thought of as outputs. An output is what you "get" from code. For example the output of `print('hello world')` is "hello world"

Only two libraries are needed for this tutorial. The first input will be NumPy (for the matrix work) and one function from SciPy (for the p-value).

## The scenario

Pretend your local university library shared a small inventory. The table below summarizes that library by counting books by cover color (on the rows) and genre (on the columns). The question you want to answer is short to ask and surprisingly tricky to "test" in a scientific way:

> Does a book's cover color tell you anything about genre, or are the the color of a book's cover and its genre unrelated?

The chi-square test of independence answers questions of this type. It starts from the assumption that the two variables are unrelated (which is the null hypothesis). The process is then to look for evidence that conflicts with the null hypothesis. If there is strong contradictory evidence, you reject the null hypothesis assumption. If the evidence is weak, you fail to reject it.

|       | Business | Science | Literature | Children's | Travel | Cook |
|-------|----------|---------|------------|------------|--------|------|
| Red   |  9       | 11      | 11         | 11         | 11     | 10   |
| Green |  9       | 10      |  9         | 10         |  9     |  9   |
| Yellow| 11       | 11      |  9         |  9         |  9     | 11   |
| Purple| 10       |  9      | 11         | 11         | 11     |  9   |
| Black | 10       | 10      |  9         | 10         |  9     |  9   |
| Other | 11       | 10      | 10         |  9         | 10     |  9   |

The numbers are fictional and were chosen to sit near ten per cell. If color and genre had no relationship, every cell should look about the same. Whether the small deviations you see are real or just noise is exactly what the chi-square test is designed to help you decide.

---

Follow this tutorial by copying the Markdown and code cells into your own notebook. The process of copying and pasting the code will help you learn the code. This will also further aclimate you to Jupyter notebooks.

## Step 1 — Enter the observed data

Markdown cell:

```markdown
Store the table from above as a Python list of lists. Each inner list is one row (one color). Each position inside a row is a genre. Keep the column order consistent with the table above.
```

Code cell:

```python
# Store the observed data as a Python list of lists
observed = [
    [ 9, 11, 11, 11, 11, 10],  # Red
    [ 9, 10,  9, 10,  9,  9],  # Green
    [11, 11,  9,  9,  9, 11],  # Yellow
    [10,  9, 11, 11, 11,  9],  # Purple
    [10, 10,  9, 10,  9,  9],  # Black
    [11, 10, 10,  9, 10,  9],  # Other
]

# Evaluate the observed data
observed
```

> Reminder that the term "evaluate" has a specific meaning when programming. In this context, it means to display the value of the variable `observed` in the output cell. Similarly we might also have said we "inspect the `observed` object by evaluating it."

You should see the same six rows of six numbers, printed back as a list of lists. That is the observed table you will be working with for the rest of the tutorial. We call this the observed table or observed matrix because it documents the observations we make from our local library.

---

## Step 2 — Inspect a single cell, a row, and a column

Markdown cell:

```markdown
A little hands-on indexing pays off later. Try a few lookups now.
```

Code cell:

```python
# Inspect individual cells, rows, and columns
red_books        = observed[0]      # the entire first row
purple_cookbooks = observed[3][5]   # row 3 (Purple), column 5 (Cook)

# Print, evaluate, inspect the results
print(f"All red books per genre: {red_books}")
print(f"The sum of all red books: {sum(red_books)}")
print(f"Purple cookbooks: {purple_cookbooks}")
```

You should see:

```
All red books per genre: [9, 11, 11, 11, 11, 10]
The sum of all red books: 63
Purple cookbooks: 9
```

Notice the two-step indexing: `observed[row][column]`. A list of lists does not let you write `observed[3, 5]` with a comma. That comma syntax is one of the conveniences NumPy will give you in Step 5.

A note for future reference is that it is difficult to get the sum of all books in the entire table using this approach. Likewise it is difficult to get the sum of all books in a single column (genre) using this approach. We solve both of these problems further below in this tutorial.

---

## Step 3 — Row totals with a `for` loop

Markdown cell:

```markdown
A natural first instinct is to walk through each row, sum its entries, and collect those sums. That is exactly what this function does. There is nothing wrong with this approach for a small table; it is short, readable, and correct.
```

Code cell:

```python
# Calculate row totals using a for loop
def row_totals_loop(matrix):
    """Sum each row of a matrix and return the totals as a list."""
    totals = []
    for row in matrix:
        totals.append(sum(row))
    return totals

# Print, evaluate, inspect function results
print(f"Each row total: {row_totals_loop(observed)}")
print(f"All books count: {total_books(sum(row_totals_loop(observed)))}")
```

You should see:

```
Each row total: [63, 56, 60, 61, 57, 59]
All books count: 356
```

Walking the function line by line:

- `def row_totals_loop(matrix):` declares a function that takes one argument, expected to be a list of lists.
- `totals = []` creates an empty list. Each pass through the loop will append one number to it.
- `for row in matrix:` walks the outer list. On each pass, `row` is bound to one of the inner lists.
- `sum(row)` is the built-in `sum`. It adds up the numbers in a single list.
- `totals.append(...)` adds that one number to the end of `totals`.
- `return totals` hands the finished list back to the caller.

Hold onto how this feels. Much simpler code will do the same jobs in Step 6.

> **Aside — f-strings**
>
> Each `print` in Steps 2 and 3 begins with `f"..."`. That `f` prefix marks an f-string (formatted string). Within the context of an f-string, any Python expression inside the squiggly brackets `{...}` is evaluated and dropped into the string in place. Compare:
>
> - `print("Each row total:", row_totals_loop(observed))` (two arguments joined by a comma).
> - `print("Each row total: " + str(row_totals_loop(observed)))` (string concatenation using the `+` operator).
> - `print(f"Each row total: {row_totals_loop(observed)}")` — one string with the value sitting exactly where it will appear in the output.
>
> The braces accept any expression, not just a bare variable name which is why `{sum(red_books)}` and `{sum(row_totals_loop(observed))}` were valid in the code you just ran. Without the `f`, `"{x}"` is three literal characters; the prefix is what turns substitution on. 
>
> Later, when learning more about f-strings, you will also see how a colon inside the braces introduces a format specification. For example, `f"{p_value:.4f}"` rounds to four decimal places, which will be useful for the p-value in Step 12. You will see f-strings again, they are the standard way to build strings everywhere in modern Python.

---

## Step 4 — Column totals with a nested `for` loop

Markdown cell:

```markdown
Column totals are where the beginner approach starts to feel awkward. A column is not a single Python list it is the *k*-th entry of every row. To collect a column you have to loop through the rows for each column, which means a loop inside a loop.
```

Code cell:

```python
# Calculate column totals using a nested for loop
def col_totals_loop(matrix):
    """Sum each column of a matrix and return the totals as a list."""
    n_cols = len(matrix[0])
    totals = []
    for c in range(n_cols):
        running = 0
        for row in matrix:
            running += row[c]
        totals.append(running)
    return totals

# Print, evaluate, inspect function results
print(f"Column totals: {col_totals_loop(observed)}")
print(f"Col 2 toal (Science): {col_totals_loop(observed)[1]}")
```

You should see:

```
Column totals: [60, 61, 59, 60, 59, 57]
Col 2 toal (Science): 61
```

Walking the function line by line:

- `n_cols = len(matrix[0])` measures the width of the table by asking the first row how long it is.
- `totals = []` is the same empty-list pattern as Step 3.
- The outer loop, `for c in range(n_cols):`, picks one column index at a time.
- `running = 0` resets an accumulator before counting that column.
- The inner loop, `for row in matrix:`, visits every row of the table.
- `running += row[c]` adds the value at column `c` of the current row to the accumulator.
- After the inner loop finishes, `totals.append(running)` records that column total.
- `print(f"Column totals: {col_totals_loop(observed)}")` prints the column totals.
- `print(f"Col 2 toal (Science): {col_totals_loop(observed)[1]}")` prints the second column total. Notice how the square bracket `[1]` actually references the second column.

Notice the shape of the work: two `for` loops, an index variable, an accumulator. The code is fine for a 6×6 table, but it is already doing extra bookkeeping that has little to do with the math. The next step shows why you might want to retire this pattern for rectangles of numbers.

---

## Step 5 — Bring in NumPy

Markdown cell:

```markdown
So far you have only used built-in Python. Time to introduce one library: NumPy.

NumPy is the standard array library in Python data science. You will reach for it whenever you have a rectangle of numbers and want to do arithmetic on the whole thing at once. The same skills carry directly into pandas, scikit-learn, PyTorch, and JAX, so the time you invest here pays back many times over.

Two things to know about a NumPy array right now:

- It looks like a list of lists when you print it, but the whole array stores numbers in one block of memory.
- Arithmetic happens *element-wise* without any loop you write. Expressions like `arr + 1`, `arr_a - arr_b`, and `arr ** 2` operate on every cell at once.

Convert your list of lists to a NumPy array and print it.
```

Code cell:

```python
# Convert list of lists to NumPy array
import numpy as np

# Create NumPy array from list of lists
observed_arr = np.array(observed)
observed_arr
```

You should see:

```
array([[ 9, 11, 11, 11, 11, 10],
       [ 9, 10,  9, 10,  9,  9],
       [11, 11,  9,  9,  9, 11],
       [10,  9, 11, 11, 11,  9],
       [10, 10,  9, 10,  9,  9],
       [11, 10, 10,  9, 10,  9]])
```

The numbers are the same. The container is different. As a small payoff, try `observed_arr[3, 5]`. It returns `9` (purple-cookbook count) with one set of brackets and a comma. That is the NumPy version of the two-step indexing from Step 2.

---

## Step 6 — Row, column, and grand totals in one line each

Markdown cell:

```markdown
NumPy reads `axis=1` as "sum across the columns of each row" which is one total per each row. `axis=0` is "sum down the rows of each column" which is one total per each column. Drop the `axis` argument and you get the grand total of every cell.
```

Code cell:

```python
# Calculate row totals, column totals, and grand total using NumPy
row_t = np.sum(observed_arr, axis=1)
col_t = np.sum(observed_arr, axis=0)
grand = np.sum(observed_arr)

# Print, evaluate, inspect results using f-strings
print(f"Row totals:  {row_t}")
print(f"Col totals:  {col_t}")
print(f"Grand total: {grand}")
```

You should see:

```
Row totals:   [63 56 60 61 57 59]
Col totals:   [60 61 59 60 59 57]
Grand total:  356
```

Confirm these match what `row_totals_loop` and `col_totals_loop` returned earlier. Same numbers; one line each; no loops written by hand. NumPy still loops internally butit just does so in compiled code that is far faster than a classic Python loop.

---

## Step 7 — What is the *expected* matrix?

Markdown cell:

```markdown
The chi-square test starts with a question:

> If cover color and genre had no relationship at all, what would the table look like?

For any one cell of the expected matrix, the recipe is short. Multiply that cell's row total by that cell's column total, then divide by the grand total of the whole table. As code: `expected_count = (row_total * col_total) / grand_total`. The same recipe in math notation (the small `i` and `j` are just labels for which row and which column you are looking at):

$$ E_{ij} = \frac{\text{row total}_i \times \text{col total}_j}{\text{grand total}} $$

A specific example helps. The Red row crossed with the Science column has a row total of `63`, a column total of `61`, and a grand total of `356`. So the expected count for that cell is `(63 * 61) / 356`, which works out to about `10.8` books.

Why does this recipe make sense? If cover color and genre have nothing to do with each other, the count you would expect in any one cell depends only on how big its row is and how big its column is. Bigger rows and bigger columns mean bigger expected counts. Dividing by the grand total keeps the answer on the same scale as the original observed counts.

A cell where the observed count is much bigger or smaller than this expected count is a potential bit of evidence weighing against the no-relationship assumption.
```

> ### New LaTeX Syntax Here
>
> LaTeX is a typesetting system commonly used for mathematical and scientific documents. It allows for the creation of complex mathematical expressions and equations. In Jupyter notebooks, LaTeX can be used to display mathematical notation in markdown cells.
>
> The `$` and `$$` delimiters are used to delimit LaTeX expressions in markdown cells. Single `$` delimiters are used for inline expressions, while double `$$` delimiters are used for block expressions.

---

## Step 8 — Expected matrix with nested `for` loops

Markdown cell:

```markdown
The literal translation of the formula above is a loop inside a loop. For every row index, for every column index, compute one expected value. That is what the beginner instinct says, so write it that way first.
```

Code cell:

```python
# Calculate expected matrix using nested for loops
def expected_loop(row_t, col_t, grand):
    """Calculate expected matrix using nested for loops."""
    expected = []
    for i in range(len(row_t)):
        row = []
        for j in range(len(col_t)):
            row.append(round(
                row_t[i] * col_t[j] / grand, 3))
        expected.append(row)
    return expected

# Print, evluate, inspect expected matrix
expected_loop_result = expected_loop(row_t, col_t, grand)
print(expected_loop_result)
```

You should see a 6×6 list of floats, every one of them close to 10.

Walking the function line by line:

- `expected = []` is the outer container for the whole result.
- `for i in range(len(row_t)):` iterates over row indices, `0` through `5`.
- `row = []` resets a fresh inner list at the start of every row.
- `for j in range(len(col_t)):` iterates over column indices, `0` through `5`.
- `row_t[i] * col_t[j] / grand` is the formula straight from Step 7.
- `row.append(...)` adds that one expected value to the current row.
- `expected.append(row)` saves the completed row into the outer container.
- `return expected` returns the finished list of lists.

Again, there is nothing wrong with this code that uses the `for` loop approach. But you are writing the same shape (outer loop, inner loop, append) for the third time. That repetition is a strong signal to you as a programmer that abother apprach may be more efficient. In this case indeed, NumPy is about to remove the need for this `for` loop approach.

---

## Step 9 — Expected matrix in one line of NumPy

Markdown cell:

```markdown
NumPy ships a function called `np.outer` whose entire job is to take two 1-D arrays and produce the table of all pairwise products. That table is exactly the numerator of the expected-value formula.
```

Code cell:

```python
# NumPy approach using np.outer
expected = np.outer(row_t, col_t) / grand
expected.round(3)
```

You should see a 6×6 NumPy array of floats, all close to 10.

Walking the line:

- `np.outer(row_t, col_t)` builds a 6×6 array where entry `(i, j)` is `row_t[i] * col_t[j]`.
- `/ grand` divides every entry of that array by the grand total. Division between an array and a single number applies element-wise without any loop you have to write.
- `.round(3)` rounds each value to 3 decimal places for cleaner display.

Reading two grids of floats and confirming they match is no fun, so let NumPy do it.

Markdown cell:

```markdown
As a matter of offering a complete range of options another approach would be to use the traditional operators `*` and `/` directly on the arrays.
```

Code cell:

```python
# Alternative approach using traditional operators
expected = (row_t.reshape(-1, 1) * col_t) / grand
expected.round(3)
```

Again, you should see a 6×6 NumPy array of floats, all close to 10 (the results here should match the results from above that used `np.outer`).

Code cell:

```python
# Verify the results match the loop-based approach
np.allclose(
    expected.round(3), 
    np.array(expected_loop(row_t, col_t, grand)))
```

You should see `True`.

`np.allclose` is the right tool for comparing arrays of floats. Never use `==` on floats because tiny rounding differences will trip you up.

---

## Step 10 — The normalized error matrix

Markdown cell:

```markdown
Now we compare observed to expected, cell by cell. The chi-square test does this in a specific shape:

$$ Z_{ij} = \frac{(O_{ij} - E_{ij})^2}{E_{ij}} $$

Squaring makes positive and negative deviations contribute equally (a cell that is two books low matters the same as a cell that is two books high). Dividing by the expected value puts each deviation on a relative scale. Being two off when ten were expected is a bigger deal than being two off when a thousand were expected.

The beginner version is yet another nested loop. The matrix version is a single line.
```

Code cell:

```python
# Calculate normalized error matrix
normalized_error = (observed_arr - expected) ** 2 / expected
normalized_error.round(3)
```

You should see a 6×6 NumPy array of small non-negative floats.

Walking the line:

- `observed_arr - expected` subtracts the two arrays *element by element*. No loop, no index variables, no accumulator. This is the heart of why matrix code is short.
- `** 2` raises every element of the resulting array to the power 2.
- `/ expected` divides every element by the matching entry of the expected matrix.

You just executed what would have been three nested loops worth of arithmetic in a single expression.

---

## Step 11 — Sum the normalized errors

Markdown cell:

```markdown
Add up every entry of the normalized-error matrix. That single number is the chi-square statistic. It is a one-number summary of how far the observed table sits from the no-relationship expectation.
```

Code cell:

```python
# Calculate chi-square statistic
chi2_stat = np.sum(normalized_error)
print(f"Chi-square statistic: {chi2_stat:.8f}")
```

- The `:.8f` formats the number to 8 decimal places.
- `np.sum()` sums all elements of the array.

You should see a small floating-point number, well under 10. Small is the right answer here. The observed table was built to look close to flat, so the per-cell deviations are tiny and their sum is too.

> ### Compare + Contrast `:.8f` vs `round(3)`
> 
> The `:.8f` format specifier is used in f-strings to format a number as a floating-point number with 8 decimal places. The `round(3)` function rounds a number to 3 decimal places. The difference is that `:.8f` is a string formatting operation, while `round(3)` is a mathematical operation. An important implication of useing `round()` is that if the result is assigned back to itself there will be information loss.

---

## Step 12 — From the statistic to a p-value

Markdown cell:

```markdown
A raw chi-square number is hard to interpret without a yardstick. The yardstick is the chi-square distribution, parameterized by a single integer called the degrees of freedom. For a contingency table:

$$ df = (\text{number of rows} - 1) \times (\text{number of columns} - 1) $$

The p-value is the probability of seeing a chi-square statistic at least as large as the one you got (or larger) if the null hypothesis (no relationship) was true. A small p-value is evidence against the null. SciPy has the distribution built in. This is the only second-library import in the tutorial.
```

Code cell:

```python
# Calculate p-value with scipy.stats
from scipy.stats import chi2

# Calculate degrees of freedom
n_rows, n_cols = observed_arr.shape
df = (n_rows - 1) * (n_cols - 1)

# Calculate p-value
p_value = chi2.sf(chi2_stat, df)

# Print, inspect, evaluate with f-strings
print(f"chi-square statistic: {chi2_stat}")
print(f"degrees of freedom:   {df}")
print(f"p-value:              {p_value}")
```

You should see `df` of `25` and a p-value much higher than `0.05`.

Walking the new pieces:

- `observed_arr.shape` is a tuple of the array's dimensions, in this case `(6, 6)`. Unpacking it into `n_rows, n_cols` is a Python pattern worth getting comfortable with.
- `chi2.sf(chi2_stat, df)` is the *survival function* of the chi-square distribution: it returns `1 - cdf(chi2_stat, df)`. It is more numerically accurate than computing `1 - chi2.cdf(...)` by hand, especially when the right tail is far out.

A high p-value means, the observed closely matches the expected. We have essentially no evidence against the no-relationship assumption. Otherwise stated we have very little evidence in favor of the alternate. Given the available evidence you fail to reject the null hypothesis.

---

## Step 13 — Wrap the whole pipeline as one function

Markdown cell:

```markdown
Each piece works. Now compose them. This is functional style: a single function with no shared state and a single job (run the test on one observed table) built out of the named operations you already understand.
```

Code cell:

```python
# Wrap the whole pipeline as one function
def chi_square_test(observed_list):
    """
    Calculate the chi-square statistic, degrees of freedom, and p-value
    for a given observed contingency table.
    
    Parameters:
    observed_list (list): A 2D list representing the observed frequencies.
    
    Returns:
    tuple: A tuple containing the chi-square statistic, degrees of freedom, and p-value.
    """
    observed_arr = np.array(observed_list)

    row_t = np.sum(observed_arr, axis=1)    # Sum along rows
    col_t = np.sum(observed_arr, axis=0)    # Sum along columns
    grand = np.sum(observed_arr)            # Grand total

    expected         = np.outer(row_t, col_t) / grand
    normalized_error = (observed_arr - expected) ** 2 / expected
    chi2_stat        = np.sum(normalized_error)

    n_rows, n_cols = observed_arr.shape    # Get the dimensions of the array
    df = (n_rows - 1) * (n_cols - 1)       # Calculate degrees of freedom
    p_value = chi2.sf(chi2_stat, df)       # Calculate p-value

    return chi2_stat, df, p_value          # Return the results

# Print, inspect, evaluate results
chi_square_test(observed)
```

You should see a tuple of three numbers including the chi-square statistic, the degrees of freedom (`25`), and the p-value. They should match what you produced one step at a time in Steps 11 and 12.

Compare the length of this function to what it would be if every step inside it were the nested-loop version. The contrast in code volume and complexity is the entire lesson of the tutorial. Using NumPy is a much more readable coding solution. Since it is more readable it is also more reproducible and maintainable (valuable features for scientific research).

---

## Step 14 — Sanity check with a known case

Markdown cell:

```markdown
Before trusting a function on real data, always run it on data whose answer you can verify by hand. Here is a deliberately lopsided 2×2 table.
```

Code cell:

```python
# Define a 2x2 table with a strong pattern
strong_pattern = [
    [30, 10],
    [10, 30]]

# Run the chi-square test
chi_square_test(strong_pattern)
```

By hand: row totals are `[40, 40]`, column totals are `[40, 40]`, the grand total is `80`, so every expected value is `40 * 40 / 80 = 20`. Each of the four cells contributes `(±10)**2 / 20 = 5`, so the chi-square statistic is `20`. Degrees of freedom is `(2 - 1) * (2 - 1) = 1`.

You should see `(20.0, 1, ~7.74e-06)`. The first two numbers match the by-hand calculation exactly. The tiny p-value says: this table looks like there is a real relationship between rows and columns.

A second sanity check. A table where observed equals expected exactly should give a chi-square of `0` and a p-value of `1`.

Code cell:

```python
# Define a 2x2 table where observed equals expected
flat = [       # No relationship between rows + columns
    [10, 10],
    [10, 10]]

# Run the chi-square test
chi_square_test(flat)
```

You should see `(0.0, 1, 1.0)`.

If either sanity check disagrees with the numbers above, something earlier in the pipeline is wrong. That is exactly what sanity checks are for.

---

## Where to go next

These extensions are in roughly increasing difficulty. Pick the ones that look interesting and ignore the rest.

1. Modify `chi_square_test` so it also returns the `expected` array and the `normalized_error` matrix. Then write a one-liner that prints the (row, column) of the single cell that contributes most to the statistic. This is how you find *which* cells are driving a significant result.
2. Replace `np.outer(row_t, col_t)` with broadcasting: `row_t[:, None] * col_t[None, :]`. Use `np.allclose` to confirm the two expressions agree. Broadcasting is the more general tool and shows up everywhere in NumPy, pandas, PyTorch, and JAX. Knowing it well is one of the highest-leverage skills in scientific Python.
3. Compare your answer to `scipy.stats.chi2_contingency(observed)`. You should match its first two return values to many digits. Reading that function's docstring is a good next exercise in itself.
4. Generate a 6×6 table with `np.random.randint` and run your function on it. How extreme do the counts have to be before the p-value drops below `0.05`? Try several seeds — `np.random.seed(0)`, `np.random.seed(1)`, and so on — to get a feel for how much p-values move around even when nothing is really going on.
5. Plot the normalized-error matrix as a heatmap with `matplotlib.pyplot.imshow` so cells that contribute most to the statistic light up visually. This is a fast way to read where a real relationship lives in a larger table.

A complete, runnable version of every step lives in `chi2-solution.ipynb`. Use it to check your work, not to skip ahead.
