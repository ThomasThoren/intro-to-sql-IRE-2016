# Intro to SQL

This guide was made for the [hands-on SQL workshop](http://www.ire.org/events-and-training/event/2199/2634/) at IRE 2016 in New Orleans. It uses bridge inspection data from the Federal Highway Administration's Bridge Inventory Database. You can [download the Louisiana data here](https://raw.githubusercontent.com/ThomasThoren/intro-to-sql-IRE-2016/master/bridge15_la.csv), courtesy of IRE.

The rest of the hands-on SQL class material is located here: [http://bit.ly/hands_on_sql_ire2016](http://bit.ly/hands_on_sql_ire2016)

## Selecting your data

__`SELECT` and `FROM`__

The two most basic SQL commands are `SELECT` and `FROM`. For every SQL query, you will always need to state which columns you want and the table where those columns are located.

Select all columns from the `bridge15_la` table. The `*` is a wildcard character, which means everything in this context.

```sql
SELECT *
FROM bridge15_la;
```

Now let's select specific columns from the `bridge15_la` table. These columns are the bridge status, sufficiency rating number, feature, structure, average daily traffic, year the bridge was built and the most recent inspection date. The table includes many more columns, but these are the columns that we will focus our attention on.

```sql
SELECT stat, suffrtno, feature, strcture, avdayno, year, inspdate
FROM bridge15_la;
```

If the column names aren't clear or are difficult to remember, you can rename the column headers to make your output easier to understand. Use the `AS` command to rename a column.

```sql
SELECT feature,
       stat AS status,
       year AS year_built,
       avdayno AS average_daily_traffic,
       suffrtno AS sufficiency_rating_number,
       strcture AS structure,
       inspdate AS most_recent_inspection_date
FROM bridge15_la;
```

We'll stick with the spreadsheet's default column names, but remember this trick if you ever get bogged down in too many abbreviations.

Try searching for various columns to explore the data. You can rearrange the order of column names in the `SELECT` statement to your liking.

## Filtering the data

__`WHERE`__

Now we're going to look at one of the most useful parts of SQL. The `WHERE` command lets you filter your data based on any number of criteria. It a row matches the given criteria, that row is returned.

For example, you could limit the rows to only those that are in Orleans Parish using the following query. The output will include all rows where the statement `cnty = '071'` is true.

```sql
SELECT *
FROM bridge15_la
WHERE cnty = '071';
```

In addition to equals (`=`), other common comparison operators include does not equal (`!=`), greater than (`>`), less than (`<`), greater than or equal (`>=`) and less than or equal (`<=`). Of course, your data must be numeric in order to use mathematical operators such as `>` or `<`. That wouldn't make much sense when comparing two words or phrases.

```sql
SELECT *
FROM bridge15_la
WHERE suffrtno < 50;
```

__`LIKE`__

Perhaps you only know a part of the text that you are seeking. SQL offers a useful command that lets you search by pieces of text. The `%` here acts as a wildcard character, meaning it can represent zero or more characters. In other words, any `feature` cells starting with "I-10" will match this filter.

```sql
SELECT *
FROM bridge15_la
WHERE feature LIKE 'I-10%';
```

You could also use it to only show features that end in "I-10."

```sql
SELECT *
FROM bridge15_la
WHERE feature LIKE '%I-10';
```

And if you wanted to find "I-10" anywhere, you could use the wildcard `%` at both the start and the end.

```sql
SELECT *
FROM bridge15_la
WHERE feature LIKE '%I-10%';
```

__`AND` and `OR`__

You might have noticed that other features refer to I-10 as "I10" or "I 10." To capture those rows as well, we can add additional filters using the `OR` condition.

```sql
SELECT *
FROM bridge15_la
WHERE feature LIKE '%I-10%'
   OR feature LIKE '%I 10%'
   OR feature LIKE '%I10%';
```

As long as the `feature` value matches at least one of those patterns, that row will be returned. With `OR`, rows are returned as long as they match at least one of the filters.

According to the [data's documentation](http://ire.org/media/uploads/files/datalibrary/samplefiles/National%20Bridge%20Inventory/readme_bridges_1.txt), a bridge with `stat` equal to 1 is structurally deficient and a bridge with `stat` equal to 2 is functionally obsolete. Let's find those bridges using two filters combined with the `OR` operator.

```sql
SELECT feature, stat, suffrtno
FROM bridge15_la
WHERE stat = '1' OR stat = '2';
```

You can accomplish the same query using the `IN` operator, as shown below. This query says that as long as the `stat` value is somewhere in that list of values, the row is a match and will be returned. I find this syntax a little bit easier to manage once you start searching for more than a few values. The downside to using `IN` is that you can no longer make use of `LIKE`, so only use `IN` when you have full-text matches.

```sql
SELECT feature, stat, suffrtno
FROM bridge15_la
WHERE stat IN ('1', '2');
```

The [data documentation](http://ire.org/media/uploads/files/datalibrary/samplefiles/National%20Bridge%20Inventory/readme_bridges_1.txt) also states that, of those bridges with `stat` equal to 1 or 2, any bridge that also has a sufficiency rating (`suffrtno`) less than 50 is eligible for replacement or rehabilitation. These are the worst of the worst bridges.

To check if a bridge is structurally deficient or functionally obsolete __and__ has a low sufficiency rating, we can use the `AND` operator.

```sql
SELECT feature, stat, suffrtno
FROM bridge15_la
WHERE (stat = '1' OR stat = '2') AND suffrtno < 50;
```

This means that a row will only be returned if the `stat` field is either "1" or "2", while also having a sufficiency rating below 50. The `OR` operator only requires one true value, but the `AND` operator requires true values from all comparisons.

Notice the use of parentheses around the `OR` operator in the above query. This groups the result of that comparison, which is then used in the `AND` comparison. The parentheses help to stay organized.

## Sorting your data

__`ORDER BY`__

Since we're exploring the worst bridges, it might help to rank those bridges from worst to best. This is where the `ORDER BY` operator helps. You can select the column which will determine the order of the rows.

This query orders the results in descending order, based on the `suffrtno` values.

```sql
SELECT feature, stat, suffrtno
FROM bridge15_la
WHERE (stat = '1' OR stat = '2') AND suffrtno < 50
ORDER BY suffrtno DESC;
```

The default setting for `ORDER BY` is ascending order. In the above query, ascending order could be achieved by writing either `...ORDER BY suffrtno ASC` or `...ORDER BY suffrtno`.

__`LIMIT`__

The `LIMIT` command forces your query to only return the specified number of rows. This is commonly used in conjunction with `ORDER BY` to show a small set of ranked rows ("The 10 worst bridges in Louisiana").

This query orders the results in ascending order (the default order), based on the `suffrtno` values, and only returns the first 10 values. Because we are sorting from low to high (bad to good) sufficiency rating numbers, and limiting the results to the first 10, this query returns the 10 worst bridges (at least according to these columns).

```sql
SELECT feature, stat, suffrtno
FROM bridge15_la
WHERE (stat = '1' OR stat = '2') AND suffrtno < 50
ORDER BY suffrtno
LIMIT 10;
```

## Aggregate functions

SQLite offers built-in functions to perform basic calculations on your data. `COUNT`, `MAX`, `MIN`, and `AVG` are some common ones. You can read more about them here: https://www.sqlite.org/lang_aggfunc.html

__`COUNT`__

Return the number of rows matching your query. This is especially useful when combined with `WHERE` statements to understand how many rows match your filters.

```sql
SELECT COUNT(*)
FROM bridge15_la;
WHERE cnty = '071';
```

__`AVG`__

Return the average value for the column specified.

```sql
SELECT AVG(suffrtno)
FROM bridge15_la;
```

__`MAX`__

Return the greatest value for the column specified.

```sql
SELECT MAX(suffrtno)
FROM bridge15_la;
```

__`MIN`__

Return the smallest value for the column specified.

```sql
SELECT MIN(suffrtno)
FROM bridge15_la;
```

## A few more notes

__Formatting__

The capitalization of SQL syntax words is not necessary, but helps to differentiate between SQL commands and other information. I find it easier to scan this way too. This is also why I include the new lines for each successive SQL command. They make reading easier but are not necessary.

The semicolon at the end of each command is not required by all SQL software, but it is by many so it's a good habit to get into.

__Comments__

As your queries grow more and more complex, it might help to write comments within your SQL code to note what a particular line does or explain why you are writing a query in the first place. Your future self with be grateful when you revisit your code.

If you are familiar with other programming languages, then you are probably familiar with the idea of comments in your code. These are lines that are not executed and only exist for people reading the code.

In SQL, you can write comments in two ways. For a single line, you can use two hyphens (`--`) to begin your comment. For example:

```sql
SELECT stat, suffrtno  -- suffrtno stands for "sufficiency rating number"
FROM bridge15_la;
```

Another more flexible way to write comments is using the `/* Comments here. */` syntax. These can be used for a single line or multiple lines. For example:

```sql
/*
Everything inside here is a comment and won't be executed in the SQL query.

This query accomplishes two things:
  - Filters the data to only include bridges labeled "structurally deficient" or "functionally obsolete."
  - Filters the data even further to only include the remaining bridges that also have sufficiency ratings below 50.
*/

SELECT feature, stat, suffrtno  -- suffrtno is the sufficiency rating number
FROM bridge15_la
WHERE (stat = '1' OR stat = '2') AND suffrtno < 50;  /* These are some bad bridges. */
```

__Data types__

Stay aware of the different types of data in your tables. Common types include integers (whole numbers), floats (numbers with decimals), booleans (True or False), text and dates.

This is very important when you have data containing a leading zero (e.g. zip code 07712). If you were to convert that to an integer (7,712), it would lose its meaning. Conversely, you should make sure numeric data is stored as numbers and not text so that you can make use of mathematical operators such as `=`, `<` and `>`.

__Differences in SQL syntaxes__

The various flavors of SQL (SQLite, MySQL, PostgreSQL, SQL Server, etc.) all have slightly different syntaxes, but they are mostly the same when it comes to basic usage. This can be annoying when switching between the SQL languages, but the good news is that they all have been around for decades. That means most syntax fixes are well-documented and only a quick Google search away.

__`NULL`__

One confusing point with SQL and programming languages in general is the idea of `NULL`. In databases, you can declare whether or not a column allows `NULL` entries, meaning whether or not they can lack any values. This is a subtle but significant difference between an empty value. An empty value means the emptiness is reported, whereas a `NULL` value means nothing is reported at all. It is the lack of anything.

You can filter based on whether a cell is `NULL` or not using `IS` or `IS NOT` as the comparison operators, instead of `=` or `!=`. Again, this is because `NULL` is not really equal to anything; it's the absence of any value.

```sql
SELECT *
FROM bridge15_la
WHERE stat IS NULL;
```

## Next steps

Once you are comfortable with these commands, try using the `GROUP BY` and `JOIN` commands. `GROUP BY` is a powerful command that lets you aggregate similar data and answer questions such as, "For each county, what is the worst bridge?" This is similar to Excel's Pivot Tables.

`JOIN` allows you to link different tables in your database, which is helpful when you are trying to link two separate data sets. For example, imagine you had one table `bridges` that listed every bridge's inspection rating and you also have a table `school_bus_routes` that has data on all school buses and the bridges on which they travel. You could perform all of the queries listed above on the `bridges` table and then join those results on the `school_bus_routes` table to see whether school buses drive over the worst-rated bridges. ([They do.](http://cu-citizenaccess.org/2013/06/05/long-delays-for-funding-plague-bridge-repairs/))

## Further reading

- https://github.com/taggartk/2016-NICAR-Adv-SQL/blob/master/SQL_queries.md
- https://github.com/tthibo/SQL-Tutorial
- https://github.com/eklucas/NICAR-Adv-SQL
- https://github.com/taggartk/2016-NICAR-Adv-SQL
- https://github.com/anthonydb/advanced-sql-nicar15