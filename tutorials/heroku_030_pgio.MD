# Overview  
This tutorial builds on the previous Heroku PostgreSQL tutorial.  You will create an application with PostgreSQL back end.  Additionally, you will read records from a local file to insert into the database and export database records to a file.  

# Application Set Up  
High level steps, if necessary, refer to previous tutorials  
1. Create a repository in GitHub:  
   Name:  **Herokupgio**  
   Add .gitignore Node  
   
2. Clone the repository to your local machine  

3. Heroku  
   1. Deploy the application from GitHub
   2. Add a Postgres Database  
   
4. Local Machine:  
   1. Add the remote Heroku git URL to your local environment.  
   
5. Initialize the Node application  
   ```npm init```  
   Add the following to the package.json file, scripts section (before the ```"test": ...``` line    
   ```"start": "node index.js",```  
   
 6. Add the following packages to the application (Express, EJS, and PostgreSQL):  
   `npm install express ejs pg`  
   
 7. Create the **index.js** start file.  Initial contents:  
    ```js
    // Add required packages
    const express = require("express");
    const app = express();

    // Set up EJS
    app.set("view engine", "ejs");


    // Start listener
    app.listen(process.env.PORT || 3000, () => {
        console.log("Server started (http://localhost:3000/) !");
    });

    // Setup routes
    app.get("/", (req, res) => {
        //res.send ("Hello world...");
        res.render("index");
    });
    ```  
    
 8. Create the **views** directory  
 
 9. Create the following file in the views directory  
    File name:  **index.ejs**  
    Content:  
    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Asher's Heroku Postgres Application</title>
        <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.5.3/dist/css/bootstrap.min.css" integrity="sha384-TX8t27EcRE3e/ihU7zmQxVncDAy5uIKz4rEkgIXeMed4M0jlfIDPvg6uqKI2xXr2" crossorigin="anonymous">
    </head>
    <body>
        <h1>Heroku PostgreSQL Application</h1>
    </body>
    </html>
    ```
 10. Start the application  
     ```npm start```  
     Navigate to the following  
     [http://localhost:3000](http://localhost:3000)  
     Confirm that your application is running  
     
# Database Connection  
 1. From the terminal, connect to the database  
     ```heroku pg:psql```  
     
 2. Run the script below to create a PRODUCT table and populate with records  
    Create a script file, content:  
     ```sql
    -- Drop the table
    DROP TABLE PRODUCT;

    -- Create table
    CREATE TABLE PRODUCT (
        prod_id    	SERIAL PRIMARY KEY,
        prod_name  	VARCHAR(20) NOT NULL,
        prod_desc 	VARCHAR(50),
        prod_price  NUMERIC
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
    
    Run the script (use your path and script file name):  
    ```\i yourPath/yourScript.sql```  
    
    Note:      
    - Although prod_id is serial (sequential), you can still manually enter a value during insert  
    - This could cause a collision  
    
 3. Add database information to the **index.js** file (below current required package).  
    ```js
    // Add database package and connection string (can remove ssl)
    const { Pool } = require('pg');
    const pool = new Pool({
      connectionString: process.env.DATABASE_URL,
      ssl: {
        rejectUnauthorized: false
      }
    });
    ```  
    
    Note:  Above uses a DATABASE_URL variable.  It needs to be defined.  
    
 4. DATABASE_URL variable  
    - You could have different values for different environments.  In our case, we will connect to Heroku's PostgreSQL cloud database.  The value will be declared in the .env file.  
    - The variable needs to be defined in order to run the application locally.  It is not required When running in Heroku PaaS: it is already set up in that environment.  
    
    1. Determine the value of DATABASE_URL  
        From the terminal (if you're connected to the database, either exit or start another integrated terminal), run  
        ```heroku config```   
        You will see the DATABASE_URL variable and value.  Copy the value (long string beginning with postgres://...)  
        
    2. Install the **dotenv** package (to manage environment variables)  
       ```npm install dotenv```  
       
    3. Configure the DATABASE_URL variable.  
       Create the following:  
       File name: **.env**  (create in the application root directory)  
       Content (use your value):  
       ```
       DATABASE_URL="postgres://abcdef123456789:12345678901234567890c674eb05bc549afe83456689206d86719dda70ccb8b6@ec2-52-200-134-180.compute-1.amazonaws.com:5432/12345678909vdq"  
       ```  
       
     4. Modify **index.js** to include the dotenv package.  Add the following line towards the top  
        ```require('dotenv').config()```  
        
     5. **.gitignore** - Note that the .env file is already included  
     
# Display Database Records  
We will start with a simple page displaying records in the **PRODUCT** table  

1. Modify **index.js** root resource ("/") as follows:  
   ```js
   app.get("/", (req, res) => {
       //res.send ("Hello world...");
       const sql = "SELECT * FROM PRODUCT ORDER BY PROD_ID";
       pool.query(sql, [], (err, result) => {
           var message = "";
           var model = {};
           if(err) {
               message = `Error - ${err.message}`;
           } else {
               message = "success";
               model = result.rows;
           };
           res.render("index", {
               message: message,
               model : model
           });
       });
   });
   ```  

   Notes:  
   - Query the database for all records in the PRODUCT table
   - Declare two variables to be sent to the rendered page:  
     - `message` - Track if query was a "success" or "Error occurred"  
     - `model` - Initially an empty object.  If query is successful, populate with resulting rows.  
 
2. Modify **index.ejs**  
   Updated Body Content:
   ```html
   <body>
       <h1>Heroku PostgreSQL Application</h1>
       <% if(message !== "success") { %>
           <h1>We Encountered an Error</h1>
           <p><%= message %></p>
       <% } else { %>
           <h2>There are <%= model.length %> products in the database</h2>
           <table class="table">
               <thead class="thead-dark">
                   <tr>
                       <th scope="col">ID</th>
                       <th scope="col">Name</th>
                       <th scope="col">Description</th>
                       <th scope="col">Price</th>
                   </tr>
               </thead>
               <tbody>
                   <% model.forEach(product => { %>
                       <tr>
                           <td><%= product.prod_id %></td>
                           <td><%= product.prod_name %></td>
                           <td><%= product.prod_desc %></td>
                           <td><%= product.prod_price %></td>
                       </tr>
                   <% }) %>
               </tbody>
           </table>
       <% } %>
   </body>
   ```  
   
   Notes:  
   - If an error occurs, the error message is displayed  
   - On success, the number of records and the data are displayed in a table (using bootstrap stylesheet)  
   
   Stop / Start the application.  Navigate to the following:  
   [http://localhost:3000](http://localhost:3000)  
   You should see the following:  
   ![Product List](images/heroku_030_pgio_table.PNG)  
   
3. Create an intentional error  
   Create an intentional error to see the results.  
   Modify **index.js**, change one line in the SQL statement as follows:  
   From: const sql = "SELECT * FROM **PRODUCT** ORDER BY PROD_ID";  
   To: const sql = "SELECT * FROM **NOPRODUCT** ORDER BY PROD_ID";  

   Stop / Start the application.  Navigate to the following:  
   [http://localhost:3000](http://localhost:3000)  
   You should see the following error:  
   ![Product List](images/heroku_030_pgio_error.PNG)  
   
   Experiment with additional errors, then fix the code.  
   
# Commit to GitHub, Deploy to Heroku  
  Complete the following  
  - Commit changes  
  - Deploy to GitHub  
  - Deploy to Heroku  
  - Validate that the application works 
  
# Add Header for Navigation / Footer  
  Create _header.ejs and _footer.ejs files.  Modify index.ejs to use the header and footer files.  
  The header file will contain application navigation bar using bootstrap stylesheet.  
  If necessary, refer to the EJS tutorial for details.  
  
1. _header.ejs file (in views directory)  
   Content:
   ```html
   <!doctype html>
   <html lang="en">

   <head>
     <meta charset="utf-8">
     <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
     <title>Asher's Heroku PostgreSQL I/O Application</title>
     <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.5.3/dist/css/bootstrap.min.css" integrity="sha384-TX8t27EcRE3e/ihU7zmQxVncDAy5uIKz4rEkgIXeMed4M0jlfIDPvg6uqKI2xXr2" crossorigin="anonymous">
   </head>

   <body>

     <div class="container">

       <nav class="navbar navbar-expand-lg navbar navbar-dark bg-dark">
         <a class="navbar-brand" href="/">Heroku Postgress IO App</a>
         <ul class="navbar-nav mr-auto">
           <li class="nav-item">
             <a class="nav-link" href="/">Products</a>
           </li>
           <li class="nav-item">
             <a class="nav-link" href="/input">Input</a>
           </li>
           <li class="nav-item">
             <a class="nav-link" href="/output">Output</a>
           </li>
         </ul>
       </nav>
   ```  
   
2. _footer.ejs file (in views directory)  
   Content:
   ```html
     <footer>
       <p>&copy; 2021 - Asher's Heroku PostgreSQL I/O Application</p>
     </footer>

   </div>

   </body>

   </html>
   ```
   
2. Update index.ejs file (in views directory)     
   Content:  
   ```html
   <%- include("_header") -%>

       <h1>Heroku PostgreSQL Application</h1>
       <% if(message !== "success") { %>
           <h1>We Encountered an Error</h1>
           <p><%= message %></p>
       <% } else { %>
           <h2>There are <%= model.length %> products in the database</h2>
           <table class="table">
               <thead class="thead-dark">
                   <tr>
                       <th scope="col">ID</th>
                       <th scope="col">Name</th>
                       <th scope="col">Description</th>
                       <th scope="col">Price</th>
                   </tr>
               </thead>
               <tbody>
                   <% model.forEach(product => { %>
                       <tr>
                           <td><%= product.prod_id %></td>
                           <td><%= product.prod_name %></td>
                           <td><%= product.prod_desc %></td>
                           <td><%= product.prod_price %></td>
                       </tr>
                   <% }) %>
               </tbody>
           </table>
       <% } %>

   <%- include("_footer") -%>
   ```  

4. Stop / Start the application.  Navigate to the following:  
   [http://localhost:3000](http://localhost:3000)  
   Confirm the changes.  
   
# Input from Local File  
  Updated to use Multer instead of express-fileupload.  
  Since we are alerady using Multer to get the forms values, we will use it for file upload.  
  [npm multer reference](https://www.npmjs.com/package/multer)  
  
  In this section, a local file will be used for inserting records to the database.  
  We cannot use typical file I/O when the application is running on the server.  
  The high-level steps are:  
  - Use Multer to upload a file to memory   
  - Look at the file content, parse it, and upload to the database  
 
1. Install Multer  
   ```npm install multer```

2. Update **index.js** to use Multer  
   Add the following content towards the top of the file  
   ```js
   const multer = require("multer");
   const upload = multer();
   ```  
   
3.  Create a view to select a file to be processed.  
    File name:  **input.ejs** (in the views directory)  
    Content:  
      ```html
      <%- include("_header") -%>

      <h1>Select a file with products for Database Insert</h1>
      <form>
          <input type="file" name="filename" required><p></p>
          <input type="submit">
      </form>
      <p id="result"></p>

      <script>
          // Typically, if the script is large, we place it in a separate file
          //   For demonstrations, it is included in the same file.
          // Handle form submission
          document.querySelector("form").addEventListener("submit", e => {
            // Cancel default behavior of sending a synchronous POST request
            e.preventDefault();
            // Create a FormData object, passing the form as a parameter
            const formData = new FormData(e.target);
            // Send form data to the server with an asynchronous POST request
            fetch("/input", {
              method: "POST",
              body: formData
            })
              .then(response => response.text())
              .then(data => {
                  const resultElement = document.getElementById("result");
                  if (data.substring(0,5) === "Error") {
                      resultElement.innerText = `Error occurred!  ${data}`
                  } else {
                      resultElement.innerText = data;
                  };
              })
              .catch(err => {
                  document.getElementById("message").textContent = `Error: ${err.message}`;
              });
          });
          </script>

      <%- include("_footer") -%>
      ```  
      
      Notes  
      - We will used AJAX rather than standard post.
      - The form is simple (you can enhance the look and feel)  
      
4. Modify **index.js**: add resources to display and process the input form  
   Content:  
   ```js  
   app.get("/input", (req, res) => {
      res.render("input");
   });

   app.post("/input",  upload.single('filename'), (req, res) => {
       if(!req.file || Object.keys(req.file).length === 0) {
           message = "Error: Import file not uploaded";
           return res.send(message);
       };
       //Read file line by line, inserting records
       const buffer = req.file.buffer; 
       const lines = buffer.toString().split(/\r?\n/);

       lines.forEach(line => {
            //console.log(line);
            product = line.split(",");
            //console.log(product);
            const sql = "INSERT INTO PRODUCT(prod_id, prod_name, prod_desc, prod_price) VALUES ($1, $2, $3, $4)";
            pool.query(sql, product, (err, result) => {
                if (err) {
                    console.log(`Insert Error.  Error message: ${err.message}`);
                } else {
                    console.log(`Inserted successfully`);
                }
           });
       });
       message = `Processing Complete - Processed ${lines.length} records`;
       res.send(message);
   });
   ```  
   
   Notes
   - Get request simply displays the form  
   - Post request  
     - Write the data to a buffer variable
     - Transform the buffer to a string and split on carriage return and line feed (\r\n) to process each line separately  
     - Split each line by comma to get each individual attribute  
     - Call pool.query to insert the record to the database 
       - This is an asynchronous update.  We do not control sequence.  
         Class discussion on alternatives.  
         
5. Stop and start the server.  Process the following file:  
   ```
   111,Envelopes,100 letter size,11.50
   112,Post-it #1,3" by 3" 1000 sheets,9.90
   113,Post-it #2,2" by 2" 1000 sheets,7.90
   114,Post-it #3,4" by 6" 1000 sheets,13.90
   ```  
   
   Notes:  
   - Save the file on your local client  
   - Choose the file from the application  
   - Submit the file for processing  
   - Validate that the records are entered in the database (connect to the database)  
   
   Image of Input View  
   ![Input](images/heroku_030_pgio_input.PNG)  
   
# Output to local file  
  In this section, a CSV file will be created from all database records and downloaded to a local file.  
  The high-level steps are:  
  - Query the database for all records  
  - Loop through the records to create a comma separated string  
  - Output the string as a CSV file  

1. Create an output view
   Filename: **output.ejs** (in the views directory)  
   Content:  
   ```html
   <%- include("_header") -%>

   <h3>Click on submit to export all database records to export.csv</h3>
   <form action="/output" method="post">
       <input type="submit" id="submit" value="Download">
   </form>
   <% if(message.substring(0,5) === "Error") { %>
       <h2>Error!</h2>
       <p><%= message %></p>
   <% } %>
   <%- include("_footer") -%>
   ```  
   
   Notes  
   - This is a simple view with a single button.  
   - It is meant to illustrate concept (you can enhance it)  
   
2. Modify **index.js**: add resources to display and process the output form  
   Content:  
   ```js
   app.get("/output", (req, res) => {
    var message = "";
    res.render("output",{ message: message });
   });


   app.post("/output", (req, res) => {
       const sql = "SELECT * FROM PRODUCT ORDER BY PROD_ID";
       pool.query(sql, [], (err, result) => {
           var message = "";
           if(err) {
               message = `Error - ${err.message}`;
               res.render("output", { message: message })
           } else {
               var output = "";
               result.rows.forEach(product => {
                   output += `${product.prod_id},${product.prod_name},${product.prod_desc},${product.prod_price}\r\n`;
               });
               res.header("Content-Type", "text/csv");
               res.attachment("export.csv");
               return res.send(output);
           };
       });
   });

   ```  
   
   Notes  
   - The Get displays the simple form  
   - The Post performs the following
     - Run a query to get all database records
     - Assigns variable output an empty string
     - Appends output with each database record
     - Sets the response header type and attachment file name (hard coded to export.csv)  
     - Returns output (will be downloaded)  
     
3. Stop and start the server.  
   - Click on the Download button and export.csv will be downloaded to your local device.  

# Commit to GitHub, Deploy to Heroku  
  Complete the following  
  - Commit changes  
  - Deploy to GitHub  
  - Deploy to Heroku  
  - Validate that the application works 
