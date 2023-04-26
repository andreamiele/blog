# Matthis Hammel Hack Solving

## How to hack a website?

Thanks to Mathis for this well-executed introductory exercise! We have access to the [website's code](https://app.coderpad.io/sandbox?question_id=247174?utm_campaign=23-Q2-Social-Twitter-TOFU-All-Global-MathisHammel&utm_source=Twitter&utm_medium=social&use_question_button), but I will explain how I managed to "hack" his site without looking at his code.

To begin with, the observation phase is important to analyze the different possible vulnerabilities. In our case, the website is composed of an integrated search bar that displays the "First name", "Last name", "Birth date" information of different people.

This search bar dynamically updates according to what is entered without needing to reload the web page. The other point is the URL, which could give us access to other pages or features of the website.

When we enter characters into the website's search bar via the console, a 200 http appears. A response to /search?query=my+search which contains a JSON file. The latter is composed of the outputs and names of three columns in the search tab.

At first glance, it is a SQL injection that seems most likely to retrieve the data that must certainly be stored in a database given the website's appearance. We will verify this and try to identify the DBMS behind it, as this may impact our future queries.

We will try to identify the number of columns in the query. Let's assume a select with the three displayed parameters or a select * which returns all columns of the table based on a where condition (e.g., where user='name').

To do this, we will close the first quote of the WHERE condition of the SQL query if it exists and add an order by 1 and --: 'order by 1--.

The quote closes the WHERE condition to make an injection. The 2 dashes are the SQL comment to nullify the rest of the query, and finally, the order by will order the result by the Xth parameter. There is indeed a WHERE condition because we have a return.

Our goal will be to increment the order by x to identify the number of columns in our query because if x>number of columns in the query, then x-1 == number of columns and may be of the table if select *.

The first column is not First name because the order is not alphabetical. Perhaps a primary key id and the SQL query a SELECT *. Order by 2 gives us the user Admin... Order by 6 returns nothing, so the query returns 5 columns or the table contains 5 columns.

Identify the string type columns: 'union select null,'a','a','a',null--. Thanks to order 1, I know that the first column is not in those displayed. The second and third are respectively First name and Last name strings, and Birthdate is not one.

The correct solution seems to be 'union select null,'a','a',null,'a'--because the first column returns nothing when I test it. Now we will try to find out what DBMS is behind the database to direct some of our queries.

We will inject into the 2nd column and sort on the 1st with null descending: 'union select null,'a',version(),null,'a' order by 1 desc--. Version() is a function of PostgreSQL that gives its version. If no return, then it is not PostgreSQL.

Now we will retrieve information from tables specific to PostgreSQL to identify different schemas with: 'UNION SELECT null, STRING_AGG(table_name, ','),'a',null,'a' FROM information_schema.tables WHERE table_schema = 'public' ORDER BY 1 desc--

Information_schema lists the information of the different tables of the DBMS according to a particular schema. To have fewer results, I filtered on the 'public' schema, which is one of the base schemas of PostgreSQL. The STRING_AGG() function allows us to concatenate strings.

We notice that the website is developed in Python with the Django web framework, given the table names, but that a table stands out: 'simplesearch_account'. Now I will try to retrieve its different column names.

Here is our new query: 'UNION SELECT null, string_agg(column_name, ','), null, null, null FROM information_schema.columns WHERE table_name = 'simplesearch_account' order by 1 desc--

Let's concatenate the columns of 'simplesearch_account' from the columns table of the PostgreSQL configuration, and BINGO! We recognize the first_name, last_name, and birth_date of our site. Id is indeed the first column on which we order the display and a password column 😊.

We are now in the home stretch of our hacking. We will now have to retrieve all the data from this table (i.e., dump the database). The next tweet is the solution.

```{SQL}
' OR ''='' UNION SELECT 1000, STRING_AGG(id||'"""'||first_name||'"""'||last_name||'"""'||password||'"""'||birth_date, '|'),'a',DATE('2023-03-01'),'a' FROM simplesearch_account ORDER BY 1 DESC-- 
```
Concatenation of columns. Rows separated by | and 3 " for the cols.

And there you have it. Now you can click on the http query to retrieve the result and find the flag of the Admin user 😊: FLAG{Congrats_y0u_pwned_it}.

Thanks for taking the time to read this. I hope this has helped you understand how to perform a SQL injection step by step! 😊 It was also my introduction because I had never done it before, so I hope you will be indulgent, and thanks to Mathis for the great exercise!