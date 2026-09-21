# [SQL-Injection-Introduction.md - Practical Lab](https://github.com/angeline-infosec/notes/blob/main/Web/SQL-Injection-Introduction.md)


## Task 9: Practical-SQL Injection 


## Level 1: Union-Based SQLi (In-Band)

What you see:  A mock browser at https://website.thm/article?id=1 showing a blog article titled "My First Article". The SQL Query box shows:

  ``` select * from article where id = 1 ```

<img width="1917" height="923" alt="image" src="https://github.com/user-attachments/assets/fc4068a3-b421-47c2-9f32-15eb361fdeaf" />


### Step 1: Find the column count. Change the id value in the URL bar:

  ``` 1 UNION SELECT 1 ```
              
<img width="1917" height="927" alt="image" src="https://github.com/user-attachments/assets/c31bc80f-adc3-4608-a62b-9d8927c1b0a6" />

Error. Wrong number of columns. Try two:

  ``` 1 UNION SELECT 1,2 ```

<img width="1917" height="922" alt="image" src="https://github.com/user-attachments/assets/29907f6d-b63e-4880-8ce4-535ee8082760" />


Still an error. Try three:

``` 1 UNION SELECT 1,2,3 ```


<img width="1917" height="925" alt="image" src="https://github.com/user-attachments/assets/3c3090f5-24ab-414d-8be9-c03f5492ab0e" />


No error, and the article loads. This means the article table has 3 columns. This is the UNION rule from Task 2: both SELECT statements must return the same number of columns. The database rejects anything that does not match, which is why each wrong guess gives you an error.

### Step 2: Make your UNION output visible. Set the article ID to 0 so the original query returns nothing:
