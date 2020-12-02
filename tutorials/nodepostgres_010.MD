# Overview  
This tutorial builds on the previous Heroku PostgreSQL tutorials.  The focus of this tutorial is a better understanding of using node.js for interfacing with PostgreSQL and asynchronous function calls.  We will not use Express.  Reference the following:  
[node-postgres website](https://node-postgres.com/)

# Application Set Up  
High level steps, if necessary, refer to previous tutorials  
1.  Crete a repository in GitHub  
    Name: **dbtests**  
    Add .gitignore Node  
   
2.  Clone the repository to your local machine  

3.  Heroku  
    1. Deploy the application from GitHub
    2. Add a Postgres Database  
   
4.  Local Machine:  
    1. Add the remote Heroku git URL to your local environment.  
   
5.  Initialize the Node application  
    ```npm init```  
    Add the following link to the package.json file, scripts section (before the ```"test": ...``` line    
    ```"start": "node index.js",```  
   
6.  Add the following packages to the application (PostgreSQL and dotenv):  
    `npm install pg dotenv`  
   
7.  .env file
    1. Create the .env file  
    2. Add a DATABASE_URL variable with the connection string information to your database  
    
8.  Setup the Database  
    Run the following script:  
    ```sql
    -- Drop the table
    DROP TABLE PRODUCT;

    -- Create table
    CREATE TABLE PRODUCT (
      prod_id    	INTEGER PRIMARY KEY,
      prod_name  	VARCHAR(20) NOT NULL,
      prod_desc 	VARCHAR(50),
      prod_price  MONEY
    );

    -- Insert records
    INSERT INTO PRODUCT (prod_id, prod_name, prod_desc, prod_price)
    VALUES 
      ('101', 'Pen', 'Black Ink', '0.99'),
      ('102', 'Pencil', 'Yellow No. 2', '0.35'),
      ('103', 'Paper', 'College Rule', '1.10'),
      ('104', 'Eraser', 'Pink', '0.75'),
      ('105', 'Calculator', 'Scientific', '99.99'),
      ('106', 'Camera', 'Desktop 720p', '39.99'),
      ('107', 'Ruler', '12"', '4.75'),
      ('108', 'Stamps', 'Dozen', '6.60'),
      ('109', 'Notepad', '8.5 by 11', '2.95'),
      ('110', 'Charger', 'USB', '22.50');
    ```  
    
# Database Commands  
1.  Create the following file:  
    Name: **index.js**  
    Content:  
    ```js
    // Add packages
    require("dotenv").config();
    // Add database package and connection string
    const { Pool } = require('pg');
    const pool = new Pool({
      connectionString: process.env.DATABASE_URL,
      ssl: {
          rejectUnauthorized: false
      }
    });
    ```  
  
2.  Simple Select / View (err, res)  
    Add the following code to **index.js**  
    ```js
    sql = "SELECT * FROM product";
    pool.query(sql, [], (err, res) => {
        console.log(err, res)
    });
    ```  
    Run the application  
    ```npm start```  
    View the Result.  
    An object is returned.  Note the following attributes:  
    - command  
    - rowCount  
    - rows - This is an Array of Objects, each object representing a database record  

    View a returned error.  Change the code as follows:  
    sql = "SELECT * FROM nonexistent";  
    Again, an object is returned.  Note the following:  
    - The first item is the error stack (message and stack detail)  
    
3.  Functions / Asynchronous  
    Replace the code with the following:  
    ```js
    const getAll = () => {
    sql = "SELECT * FROM product";
    pool.query(sql, [], (err, res) => {
        if (err) {
            return err.message;
        }
        return res.rows;
    });
    };

    const result = getAll();
    console.log(result);
    ```  
    
    Execute the updated program.  You will see the following output:  
    ```undefined```  
    Watch closely and you will notice that ```undefined``` is returned right away and the prompt eventually appears.  
    Why are we getting back ```undefined```?  
    JavaScript is single threaded.  The ```pool.query``` method is asynchronous.  So, JavaScript continues running and waits for the response (the promise) to be fulfilled later.  Modify the code to remove the (err, res) callback functions as follows:
    ```js
    const getAll = () => {
        sql = "SELECT * FROM product";
        // For illustration, not using try catch
        return pool.query(sql, []);
    };

    const result = getAll();
    console.log(result);

    ```  
    
    You see the following output:  
    ```js
    Promise { <pending> }
    ```  
    Note that a promise is pending.  
    
    Return to the original code and add ```console.log()``` statements as follows to see what is happening: 
    ```js
    const getAll = () => {
        console.log("--- STEP 2: Inside getAll() ---");
        sql = "SELECT * FROM product";
        pool.query(sql, [], (err, res) => {
            if (err) {
                console.log("--- STEP 3: Error ---");
                return err.message;
            }
            console.log("--- STEP 3: No Error ---");
            return res.rows;
        });
    };
    
    console.log("--- STEP 1: Before call to getAll() ---");
    const result = getAll();
    console.log("--- STEP 4: After call to getAll() ---");
    console.log(result);
    ```  
    
    Note the following output:  
    ```js
    --- STEP 1: Before call to getAll() ---
    --- STEP 2: Inside getAll() ---
    --- STEP 4: After call to getAll() ---
    undefined
    --- STEP 3: No Error ---
    ```  
    
    Explanation:  
    1.  Execution begins and program gets to ```console.log("--- STEP 1: Before call to getAll() ---");``` and executes it  
    2.  Then, ```getAll()``` is started and ```console.log("--- STEP 2: Inside getAll() ---");``` is executed  
    3.  Then, ```pool.query(...)``` is queued up.  It is an Asynchronous promise, so JavaScript continues to the next command  
    4.  ```console.log("--- STEP 4: After call to getAll() ---");``` is executed  
    5.  Then, ```console.log(result);``` is executed.  Note that ```pool.query(...)``` is not complete yet, so ```undefined``` is output  
    6.  Then, ```pool.query(...)``` is completed successfully and ```console.log("--- STEP 3: No Error ---")``` is executed  
    
4.  How can we run the code in sequence?  
    There are several methods.  Below are a few approaches.  
    
    1.  Callback - The asynchronous method calls the next step rather than returning a value 
        (This is the approach used in the Simple Select above)  
        The call back function has two parameters: ``` err, res```.  When the promise is fulfilled, the callback function is executed.  
        ```js
        const getAll = () => {
            sql = "SELECT * FROM product";
            pool.query(sql, [], (err, res) => {
                if (err) {
                    console.log(err.message);
                }
                console.log(res.rows);
            });
        };

        getAll();    
        ```  
    
    2.  Promise - Same as Callback, but using promise syntax (as discussed in our text)  
        ```js
        const getAll = () => {
            sql = "SELECT * FROM product";
            pool.query(sql, [])
                .then(res => {console.log(res.rows)})
                .catch(err => {console.log(err.message)});
        };

        getAll();
        ```

    3.  Promise - Caller Side
        If you really want the result back from a function call, you can use promise syntax on the caller side (when waiting for a promise).  For example:  
        ```js
        const getAll =  () => {
            console.log("--- STEP 2: Inside getAll() ---");
            sql = "SELECT * FROM product";
            // For illustration, not using try catch
            return pool.query(sql, []); 
        };


        console.log("--- STEP 1: Before call to getAll() ---");
        getAll()
            .then(result => {
                console.log(result);
            })
            .catch(err => {
                console.log(err.message);
            });
        console.log("--- STEP 4: After call to getAll() ---");
        ```  
        
        Note:  
        - If we didn't wait for getAll() to complete, it would return a promise  
        
    4.  Async Function with await  
        Another approach to waiting for results from a function call is to can create an ```async``` function and ```await``` for the asynchronous process to complete prior to continuing execution.  See the code below (using anonymous async function):  
        ```js
        const getAll =  () => {
            console.log("--- STEP 2: Inside getAll() ---");
            sql = "SELECT * FROM product";
            // For illustration, not using try catch
            return pool.query(sql, []); 
        };

        (async () => {
            console.log("--- STEP 1: Before call to getAll() ---");
            const result = await getAll();
            console.log("--- STEP 4: After call to getAll() ---");
            console.log(result);
        }
        )()
        ```
        
        See below with try catch block  
        ```js
        const getAll = async () => {
            console.log("--- STEP 2: Inside getAll() ---");
            sql = "SELECT * FROM product";
            try {
                const result =  await pool.query(sql, []);
                console.log("--- STEP 3: No Error ---");
                return (result).rows;
            } catch (err) {
                console.log("--- STEP 3: Error ---");
                return err.message;
            };
        };

        (async () => {
            console.log("--- STEP 1: Before call to getAll() ---");
            const result = await getAll();
            console.log("--- STEP 4: After call to getAll() ---");
            console.log(result);
        }
        )()
        ```  
        
5.  Get Total Records / Modularize Code  
    In this section, we'll create a function to return total number of records in the database.  We will then create a separate module/file to break up our code.  
    
    1.  Modify index.js to contain the following code  
        ```js
        const getTotalRecords = () => {
            sql = "SELECT COUNT(*) FROM product";
            return pool.query(sql)
                .then(result => {
                    return {
                        msg: "success",
                        totRecords: result.rows[0].count
                    }
                })
                .catch(err => {
                    return {
                        msg: `Error: ${err.message}`
                    }
                });
        };

        getTotalRecords()
            .then(result => {
                if (result.msg.substring(0, 5) === "Error") {
                    console.log(`Error Encountered.  ${result.msg}`);
                } else {
                    console.log(`Total number of database records: ${result.totRecords}`);
                };
            })
            .catch(err => {
                console.log(`Error: ${err.message}`);
            });
        ```  
        
        Run the program.  You should get the following output:  
        ```Total number of database records: 10```  
        
        Test it with an error (change the table name to doesnotexist and rerun).  You should get the following:  
        ```Error Encountered.  Error: relation "doesnotexist" does not exist```  
        
    2.  Modularize the code. 
        Place the database connectivity and functions in a separate file.  
        Create the following file:  
        Name: **dblib.js**  
        Content: 
        ```js
        // Add packages
        require("dotenv").config();
        // Add database package and connection string
        const { Pool } = require('pg');
        const pool = new Pool({
            connectionString: process.env.DATABASE_URL,
            ssl: {
                rejectUnauthorized: false
            }
        });

        const getTotalRecords = () => {
            sql = "SELECT COUNT(*) FROM product";
            return pool.query(sql)
                .then(result => {
                    return {
                        msg: "success",
                        totRecords: result.rows[0].count
                    }
                })
                .catch(err => {
                    return {
                        msg: `Error: ${err.message}`
                    }
                });
        };

        module.exports.getTotalRecords = getTotalRecords;
        ```  
        
        Notes:  
        - Function is now in a separate file
        - Exported the function
        
        Modify index.js as follows (entire content below):
        ```js
        // Add packages
        const dblib = require("./dblib.js");

        dblib.getTotalRecords()
            .then(result => {
                if (result.msg.substring(0, 5) === "Error") {
                    console.log(`Error Encountered.  ${result.msg}`);
                } else {
                    console.log(`Total number of database records: ${result.totRecords}`);
                };
            })
            .catch(err => {
                console.log(`Error: ${err.message}`);
            });
        ```  
        
        Notes:  
        - Added the dblib module 
        - Used getTotalRecords() function from dblib  
        
        Execute the program as usual:  ```npm start```  
        You should get the same results as in the previous step.  
        
6.  Simple Insert  
    Add an Insert function to dblib.js and use it from index.js  
    Add the following code to **dblib.js**
    ```js
    const insertProduct = (product) => {
        // Will accept either a product array or product object
        if (product instanceof Array) {
            params = product;
        } else {
            params = Object.values(product);
        };

        const sql = `INSERT INTO product (prod_id, prod_name, prod_desc, prod_price)
                     VALUES ($1, $2, $3, $4)`;

        return pool.query(sql, params)
            .then(res => {
                return {
                    trans: "success", 
                    msg: `Product id ${params[0]} successfully inserted`
                };
            })
            .catch(err => {
                return {
                    trans: "fail", 
                    msg: `Error on insert of product id ${params[0]}.  ${err.message}`
                };
            });
    };

    // Add this at the bottom
    module.exports.insertProduct = insertProduct;
    ```  
    
    1. Use an Array: Add the following code to **index.js** 
    ```js
    const producta = [200,'Book', 'The JS Way', 9.99];
    dblib.insertProduct(producta)
        .then(result => {
            if (result.trans === "fail") {
                console.log("ERROR OCCURED");
                console.log(result.msg);
            } else {
                console.log("Insert Successful");
                console.log(result.msg);
            }
        });
    ```  
    
    Execute the program.  You should see the following:  
    ```
    Insert Successful
    Product id 200 successfully inserted
    ```   
    
    Execute the program again.  You should see the following:
    ```
    ERROR OCCURED
    Error on insert of product id 200.  duplicate key value violates unique constraint "product_pkey"
    ```  
    
    Notes:
    - Query the database to confirm record was inserted  
    - Note the usage of parameters for the SQL INSERT statement  
    - You may change the array to an object and test the code (next section will use project objects)  

7.  Loop Insert / Capture Error / Asynchronous  
    In this exercise, a loop is used to insert multiple records into the database.  Errors are captured and displayed on the console.  Sequencing the inserts is reviewed.  
    The goal is to call the insert procedure multiple times and once complete, return a status summary:  
    - Number of records processed  
    - Number of records inserted  
    - Number of failed insertions  
    - Error for each failed insertion  
    
    Modify **index.js** as shown below.  **INTENTIONAL LOGIC ERROR**  
    - Create an array of project objects  
    - Create variables to store records inserted, failed, and error message  
    - Use a loop to insert the records  
    There is an intentional error below to demonstrate asynchronous behavior.  
    ```js
    // Product array
    const products = [
        {
            prod_id: 110,
            prod_name: 'Charger',
            prod_desc: 'USB',
            prod_price: 22.50
        },
        {
            prod_id: 200,
            prod_name: 'Book',
            prod_desc: 'The JS Way',
            prod_price: 9.99
        },
        {
            prod_id: 201,
            prod_name: 'Large Clips',
            prod_desc: 'Large binder clips',
            prod_price: 4.25
        },
        {
            prod_id: 202,
            prod_name: 'Medium Clips',
            prod_desc: 'Medium binder clips',
            prod_price: 3.25
        },
        {
            prod_id: 203,
            prod_name: 'Small Clips',
            prod_desc: 'Small binder clips',
            prod_price: 2.25
        }
    ];

    // Declare variables
    var numFailed = 0;
    var numInserted = 0;
    var errorMessage = "";

    // Loop to insert - Contains logic error for summary stats
    console.log("--- STEP 1: Pre-Loop");
    for (prod of products) {
        console.log("--- STEP 2: In-Loop Before Insert");
        const result = dblib.insertProduct(prod);
        console.log("--- STEP 3: In-Loop After Insert");
        console.log("result is: ", result);
        if (result.trans === "success") {
            numInserted++;
        } else {
            numFailed++;
            errorMessage += `${result.msg} \r\n`;
        };
    };    
    console.log("--- STEP 4: After-Loop");
    console.log(`Records processed: ${numInserted + numFailed}`);
    console.log(`Records successfully inserted: ${numInserted}`);
    console.log(`Records with insertion errors: ${numFailed}`);
    if(numFailed > 0) {
        console.log("Error Details:");
        console.log(errorMessage);
    };
    ```  
    
    Execute the program.  You should see the following:  
    ```
    --- STEP 1: Pre-Loop
    --- STEP 2: In-Loop Before Insert
    --- STEP 3: In-Loop After Insert
    result is:  Promise { <pending> }
    --- STEP 2: In-Loop Before Insert
    --- STEP 3: In-Loop After Insert
    result is:  Promise { <pending> }
    --- STEP 2: In-Loop Before Insert
    --- STEP 3: In-Loop After Insert
    result is:  Promise { <pending> }
    --- STEP 2: In-Loop Before Insert
    --- STEP 3: In-Loop After Insert
    result is:  Promise { <pending> }
    --- STEP 2: In-Loop Before Insert
    --- STEP 3: In-Loop After Insert
    result is:  Promise { <pending> }
    --- STEP 4: After-Loop
    Records processed: 5
    Records successfully inserted: 0
    Records with insertion errors: 5
    Error Details:
    undefined
    undefined
    undefined
    undefined
    undefined    
    ```  
    
    Notes:  
    - Notice all the pending promises.  
    - errorMessage is appended with `undefined` since the result is still pending  
    
    **Database**  
    Query the database:  
    `SELECT * FROM product;`  
    Notice that records 201, 202, and 203 were inserted.  
    Delete the inserted records:  
    `DELETE FROM product WHERE prod_id > 200;`  
    
    How can we fix this?  
    Several ways, as discussed above.  Below is one method using an async function with await.  
    Change **index.js** per the below:  
    ```js
    // Loop to insert - using async () function and await
    // Not using try catch block
    (async () => {
        console.log("--- STEP 1: Pre-Loop");
        for (prod of products) {
            console.log("--- STEP 2: In-Loop Before Insert");
            const result = await dblib.insertProduct(prod);
            console.log("--- STEP 3: In-Loop After Insert");
            console.log("result is: ", result);
            if (result.trans === "success") {
                numInserted++;
            } else {
                numFailed++;
                errorMessage += `${result.msg} \r\n`;
            };
        };    
        console.log("--- STEP 4: After-Loop");
        console.log(`Records processed: ${numInserted + numFailed}`);
        console.log(`Records successfully inserted: ${numInserted}`);
        console.log(`Records with insertion errors: ${numFailed}`);
        if(numFailed > 0) {
            console.log("Error Details:");
            console.log(errorMessage);
        };
    })()

    ```  
    
    Rerun the program (make sure you deleted the inserted database records).  You should see the following:  
    ```
    --- STEP 1: Pre-Loop
    --- STEP 2: In-Loop Before Insert
    --- STEP 3: In-Loop After Insert
    result is:  { trans: 'fail',
      msg:
       'Error on insert of product id 110.  duplicate key value violates unique constraint "product_pkey"' }
    --- STEP 2: In-Loop Before Insert
    --- STEP 3: In-Loop After Insert
    result is:  { trans: 'fail',
      msg:
       'Error on insert of product id 200.  duplicate key value violates unique constraint "product_pkey"' }
    --- STEP 2: In-Loop Before Insert
    --- STEP 3: In-Loop After Insert
    result is:  { trans: 'success',
      msg: 'Product id 201 successfully inserted' }
    --- STEP 2: In-Loop Before Insert
    --- STEP 3: In-Loop After Insert
    result is:  { trans: 'success',
      msg: 'Product id 202 successfully inserted' }
    --- STEP 2: In-Loop Before Insert
    --- STEP 3: In-Loop After Insert
    result is:  { trans: 'success',
      msg: 'Product id 203 successfully inserted' }
    --- STEP 4: After-Loop
    Records processed: 5
    Records successfully inserted: 3
    Records with insertion errors: 2
    Error Details:
    Error on insert of product id 110.  duplicate key value violates unique constraint "product_pkey"
    Error on insert of product id 200.  duplicate key value violates unique constraint "product_pkey"
    ```  
    
    Notes:  
    - Program now runs sequentially with the desired feedback.
    - In reality, you would log errors in a log file/database and let the program execute as fast as possible  
    - Execute the program again.  This time you should see the following summary:  
    ```
    Records processed: 5
    Records successfully inserted: 0
    Records with insertion errors: 5
    Error Details:
    Error on insert of product id 110.  duplicate key value violates unique constraint "product_pkey"
    Error on insert of product id 200.  duplicate key value violates unique constraint "product_pkey"
    Error on insert of product id 201.  duplicate key value violates unique constraint "product_pkey"
    Error on insert of product id 202.  duplicate key value violates unique constraint "product_pkey"
    Error on insert of product id 203.  duplicate key value violates unique constraint "product_pkey"
    ```  
# Commit to GitHub / Deploy to Heroku  
  Complete the following  
  - Commit changes
  - Deploy to GitHub
  - Deploy to Heroku
  - Validate that the application works  
  
# Conclusion  
  You should now have a better understanding of using node.js for interfacing with PostgreSQL and asynchronous function calls.  
