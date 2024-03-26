# Overview  
  This tutorial builds on the previous node-postgres tutorial.  Express and EJS are added to create a web-based application.  We will create a search screen and post back records found using both AJAX and synchronous (default) posts.  
  
  **Prerequisite**  
  Complete the previous tutorial.  
  
# Application Setup  

1.  OPTIONAL - Make a backup of the previous tutorial at the current state  
    
2.  Add the following packages  
    `npm install express ejs multer`
    
3.  Create the following directories:  
    - views  
    - public  
    
4.  Modify **index.js** file.  
    Content:  
    ```js
    // Required modules 
    const express = require("express");
    const app = express();
    const dblib = require("./dblib.js");

    const multer = require("multer");
    const upload = multer();

    // Add middleware to parse default urlencoded form
    app.use(express.urlencoded({ extended: false }));

    // Setup EJS
    app.set("view engine", "ejs");

    // Enable CORS (see https://enable-cors.org/server_expressjs.html)
    app.use((req, res, next) => {
        res.header("Access-Control-Allow-Origin", "*");
        res.header(
            "Access-Control-Allow-Headers",
            "Origin, X-Requested-With, Content-Type, Accept"
        );
        next();
    });

    // Application folders
    app.use(express.static("public"));
    
    // Start listener
    app.listen(process.env.PORT || 3000, () => {
        console.log("Server started (http://localhost:3000/) !");
    });

    // Setup routes
    app.get("/", (req, res) => {
        //res.send("Root resource - Up and running!")
        res.render("index");
    });
    ```  
    
5.  Create the following file in the views directory  
    File name: **index.ejs**  
    Content:  
    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Asher's Node PostgreSQL Application</title>
        <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH" crossorigin="anonymous">
    </head>
    <body>
        <h1>Node PostgreSQL Application</h1>
    </body>
    </html>
    ```  
    
    Start the application and confirm that the Web Server is Running  
    
6.  Header for Navigation / Footer  
    1.  _header.ejs file (in the views directory)  
        Content:  
    ```html
    <!doctype html>
    <html lang="en">

    <head>
      <meta charset="utf-8">
      <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
      <title>Asher's Node PostgreSQL Application</title>
      <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH" crossorigin="anonymous">
    </head>

    <body>

      <div class="container">

        <nav class="navbar navbar-expand-lg navbar navbar-dark bg-dark">
          <a class="navbar-brand" style="margin-left: 10px" href="/">Node PostgreSQL Application</a>
          <ul class="navbar-nav mr-auto">
            <li class="nav-item">
              <a class="nav-link" href="/">Products</a>
            </li>
            <li class="nav-item">
              <a class="nav-link" href="/search">Search Default Post</a>
            </li>
            <li class="nav-item">
              <a class="nav-link" href="/searchajax">Search AJAX</a>
            </li>
          </ul>
        </nav>
    ```  
    
    2.  _footer.ejs file (in the views directory)  
        Content (modify YYYY to current year):      
        ```html  
        <hr>
        <footer>
            <p>&copy; YYYY - Asher's Node PostgreSQL Application</p>
          </footer>

        </div>

        </body>

        </html>

        ```  
        
    3.  Modify index.ejs per the below (in the views directory)  
        Content:      
        ```html          
        <%- include("_header") -%>

        <h1>Node PostgreSQL Application</h1>

        <h2>Choose a task from the top navigation bar</h2>

        <%- include("_footer") -%>
        ```  
        
        Refresh the page and verify that the header and footer are displayed.
        
# Search Screen (default POST)  
In this section, we create a search form for the products and display the results.  
1.  Create a search page.  
    File: search.ejs (in the views directory)  
    Content (initial, will update):  
    ```html  
    <%- include("_header") -%>

    <h2> Find Products</h2>

    <h4>Total number of records in the database: XXX-Will Add Code</h4>

    <!-- Not using bootstrap, just a simple form that you may enhance -->
    <form action="/search" method="POST">
        <table>
            <tr>
                <td><label for="prod_id">ID</label>:</td>
                <td><input type="number" name="prod_id" id="prod_id" /></td>
                <td>(equal)</td>
            </tr>
            <tr>
                <td><label for="prod_name">Name</label>:</td>
                <td><input type="text" name="prod_name" id="prod_name" /></td>
                <td>(starts with)</td>
            </tr>
            <tr>
                <td><label for="prod_desc">Description</label>:</td>
                <td><input type="text" name="prod_desc" id="prod_desc" /></td>
                <td>(starts with)</td>
            </tr>
            <tr>
                <td><label for="prod_price">Price</label>:</td>
                <td><input type="text" name="prod_price" id="prod_price" /></td>
                <td>(greater than or equal)</td>
            </tr>
            <tr>
                <td span="2">
                    <input type="submit" value="Find" />
                    <input type="reset" value="Reset" />
                </td>
            </tr>
        </table>
    </form>

    <%- include("_footer") -%>   
    ```  
2.  Add a route to the search screen.    
    File: index.js  
    Add the following:  
    ```js
    app.get("/search", (req, res) => {
        res.render("search");
    });
    ```  

    Stop / Start the applications.  Validate that the search screen displays.  

3.  Modify index.js to send the totalRecords to the search.ejs view.  
    1.  index.js changes:  
        ```js
        app.get("/search", async (req, res) => {
            // Omitted validation check
            const totRecs = await dblib.getTotalRecords();
            res.render("search", {
                type: "get",
                totRecs: totRecs.totRecords
            });
        });
        ```  

        Notes:
        - Note the `async` function and `await`.  Try the code without this.  What do you think will be returned?  
        - Current code does not perform validation that dblib.getTotalRecords() was successful.  You can add that and test.  
        - Added a key=type and value="get" to the object sent to the view.  This will be used in the EJS view to determine what to render based on a "get" or a "post" response
    
    2.  search.ejs change (modify the single line below).  Display the variable passed from index.js  
        ```html
        <h4>Total number of records in the database: <%= totRecs %></h4>
        ```  
        
        Stop / Start the application and verify that the total is displayed.  
        
4.  Create a function to return database records based on a search criterion entered  
    dblib.js changes, add the following function:  
    ```js
    const findProducts = (product) => {
        // Will build the query based on data provided in the form
        //  Use parameters to avoid sql injection

        // Declare variables
        let i = 1;
        const params = [];
        let sql = "SELECT * FROM product WHERE true";

        // Check data provided and build the query as necessary
        if (product.prod_id !== "") {
            params.push(parseInt(product.prod_id));
            sql += ` AND prod_id = $${i}`;
            i++;
        };
        if (product.prod_name !== "") {
            params.push(`${product.prod_name}%`);
            sql += ` AND UPPER(prod_name) LIKE UPPER($${i})`;
            i++;
        };
        if (product.prod_desc !== "") {
            params.push(`${product.prod_desc}%`);
            sql += ` AND UPPER(prod_desc) LIKE UPPER($${i})`;
            i++;
        };
        if (product.prod_price !== "") {
            params.push(parseFloat(product.prod_price));
            sql += ` AND prod_price >= $${i}`;
            i++;
        };

        sql += ` ORDER BY prod_id`;
        // for debugging
         console.log("sql: " + sql);
         console.log("params: " + params);

        return pool.query(sql, params)
            .then(result => {
                return { 
                    trans: "success",
                    rows: result.rows
                }
            })
            .catch(err => {
                return {
                    trans: "Error",
                    error: `Error: ${err.message}`
                }
            });
    };

    // Add towards the bottom of the page
    module.exports.findProducts = findProducts;
    ```  
    
    Notes:  
    - There are several approaches to do this  
    - Variables  
      - Integer set to 1.  This will represent each parameter $1, $2, ...  
      - Params are defaulted to no parameters and we push values if data is entered  
      - sql statement is defaulted to return all records and conditions are added if data is entered

5.  Modify index.js to handle the "POST" request.  
    Modify the app.post method per the code below:  
    ```js
    app.post("/search", async (req, res) => {
        // Omitted validation check
        //  Can get totRecs from the page rather than using another DB call.
        //  Add it as a hidden form value.
        const totRecs = await dblib.getTotalRecords();

        dblib.findProducts(req.body)
            .then(result => {
                res.render("search", {
                    type: "post",
                    totRecs: totRecs.totRecords,
                    result: result,
                    prod: req.body
                })
            })
            .catch(err => {
                res.render("search", {
                    type: "post",
                    totRecs: totRecs.totRecords,
                    result: `Unexpected Error: ${err.message}`,
                    prod: req.body
                });
            });
    });
    ```  
    
    Notes:  
    - Recalculated totRecs.  Should get this as a form data and return to the form.  This is an extra database call.  
      - OK for demonstration
    - Call dblib.findProducts().  Omitted error checking.  
      - Return the type of request, totRecs, result from dblib.findProducts(), and the values sent from the form (req.body)  
        - Use req.body to repopulate the form with values initially entered.
      

6.  Modify search.ejs to display the results.  
    Below is the full content of search.ejs:  
    ```js
    <%- include("_header") -%>

    <h2> Find Products</h2>

    <h4>Total number of records in the database: <%= totRecs %></h4>

    <!-- Not using bootstrap, just a simple form that you may enhance -->
    <form action="/search" method="POST">
        <table>
            <tr>
                <td><label for="prod_id">ID</label>:</td>
                <td><input type="number" name="prod_id" id="prod_id" value="<%= prod.prod_id %>" /></td>
                <td>(equal)</td>
            </tr>
            <tr>
                <td><label for="prod_name">Name</label>:</td>
                <td><input type="text" name="prod_name" id="prod_name" value="<%= prod.prod_name %>" /></td>
                <td>(starts with)</td>
            </tr>
            <tr>
                <td><label for="prod_desc">Description</label>:</td>
                <td><input type="text" name="prod_desc" id="prod_desc" value="<%= prod.prod_desc %>" /></td>
                <td>(starts with)</td>
            </tr>
            <tr>
                <td><label for="prod_price">Price</label>:</td>
                <td><input type="text" name="prod_price" id="prod_price" value="<%= prod.prod_price %>" /></td>
                <td>(greater than or equal)</td>
            </tr>
            <tr>
                <td span="2">
                    <input type="submit" value="Find" />
                    <input type="reset" value="Reset" />
                </td>
            </tr>
        </table>
    </form>

    <!-- Omitted Error Checking / Needs to be added -->
    <% if (type === "post") { %>
        <% if (result.rows.length === 0) { %>
            <h3>No records found!</h3>
        <% } else { %>
            <table>
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Name</th>
                        <th>Desc</th>
                        <th>Price</th>
                    </tr>
                </thead>
                <tbody>
                    <% result.rows.forEach(product => { %> 
                        <tr>
                            <td><%= product.prod_id %></td>
                            <td><%= product.prod_name %></td>
                            <td><%= product.prod_desc %></td>
                            <td><%= product.prod_price %></td>
                        </tr>
                    <% }); %>
                </tbody>

            </table>
        <% } %>
    <% } %>

    <%- include("_footer") -%>   
    ```  
    
    Notes:  
    - Each input type is populated with the value sent from index.js, e.g. value="<%= prod.prod_id %>"  
    - At the bottom of the view, check if the response is a "post", if it is, display the results.  
      - Ensure that data is returned (result.rows.length === 0)  
    - Display the results in a table, iterating through the results
    - Error checking is omitted but should be added.  Experiment with bad SQL and see the results.  
    
7.  Modify index.js app.get() route  
    Since we want to retain the values the user entered in the form; we need to handle the initial request: "get"  
    There are two approaches:  
    - Send empty data from index.js  (approach shown below)
    - Check in the search.ejs view if the form is a "post" or a "get".  If it is a "get", set the values to empty data.  
    
    Content (sending empty data from index.js):  
    ```js
    app.get("/search", async (req, res) => {
        // Omitted validation check
        const totRecs = await dblib.getTotalRecords();
        //Create an empty product object (To populate form with values)
        const prod = {
            prod_id: "",
            prod_name: "",
            prod_desc: "",
            prod_price: ""
        };
        res.render("search", {
            type: "get",
            totRecs: totRecs.totRecords,
            prod: prod
        });
    });
    ```  
    
# Search Screen (AJAX)  
Perform the same task using AJAX.  Recall that with AJAX, we only update a portion of the page that changes.  So, we do not need to send form data back, since we don't change the form.  

1.  Create a new view page.  
    Name: searchajax.ejs (in the views directory)  
    Content:
    ```js
    <%- include("_header") -%>

    <h2> Find Products AJAX</h2>

    <h4>Total number of records in the database: <%= totRecs %></h4>

    <!-- Not using bootstrap, just a simple form that you may enhance -->
    <!-- Using AJAX, no form action -->
    <form>
        <table>
            <tr>
                <td><label for="prod_id">ID</label>:</td>
                <td><input type="text" name="prod_id" id="prod_id" type="number" /></td>
                <td>(equal)</td>
            </tr>
            <tr>
                <td><label for="prod_name">Name</label>:</td>
                <td><input type="text" name="prod_name" id="prod_name" /></td>
                <td>(starts with)</td>
            </tr>
            <tr>
                <td><label for="prod_desc">Description</label>:</td>
                <td><input type="text" name="prod_desc" id="prod_desc" /></td>
                <td>(starts with)</td>
            </tr>
            <tr>
                <td><label for="prod_price">Price</label>:</td>
                <td><input type="text" name="prod_price" id="prod_price" /></td>
                <td>(greater than or equal)</td>
            </tr>
            <tr>
                <td span="2">
                    <input type="submit" value="Find" />
                    <input type="reset" value="Reset" />
                </td>
            </tr>
        </table>
    </form>

    <div id="output"></div>

    <script src="searchajax.js"></script>

    <%- include("_footer") -%>   
    ```  
    
    Notes:
    - No need to set default values to form input values
    - Form does not have an action: handled by AJAX
    - Including `searchajax.js` script to handle form submit  
    
2.  Create the script file  
    Name: searchajax.js (in the public directory)  
    Content:  
    ```js
    // Function to display results
    const displayResults = (result) => {
      const divElement = document.getElementById("output");
      // Reset output for each call
      divElement.innerHTML = "";

      if (result.trans === "Error") {
          // Create h2 and paragraph elements and add to div
          const h2Elem = document.createElement("h2");
          h2Elem.innerText = "Application Error";
          const paraElement = document.createElement("p");
          paraElement.innerText = result.error;
          // Add elements
          divElement.appendChild(h2Elem);
          divElement.appendChild(paraElement);
      } else {
          if (result.rows.length === 0) {
            // Create h3 and add to div
            const h3Elem = document.createElement("h3");
            h3Elem.innerText = "No Records found!";
            divElement.appendChild(h3Elem);
          } else {
              // Create a table element and table header row
              const tblElement = document.createElement("table");
              const theadElement = document.createElement("thead");
              const thRowElement = document.createElement("tr");
              const thIdElement = document.createElement("td");
              thIdElement.innerText = "ID";
              const thNameElement = document.createElement("td");
              thNameElement.innerText = "Name";
              const thDescElement = document.createElement("td");
              thDescElement.innerText = "Desc";
              const thPriceElement = document.createElement("td");
              thPriceElement.innerText = "Price";
              // Add elements
              thRowElement.appendChild(thIdElement);
              thRowElement.appendChild(thNameElement);
              thRowElement.appendChild(thDescElement);
              thRowElement.appendChild(thPriceElement);
              //
              theadElement.appendChild(thRowElement);
              //
              tblElement.appendChild(theadElement);

              // Loop
              result.rows.forEach(product => { 
                // Create table rows
                const trElement = document.createElement("tr");
                const tdIdElement = document.createElement("td");
                tdIdElement.innerText = product.prod_id;
                const tdNameElement = document.createElement("td");
                tdNameElement.innerText = product.prod_name;
                const tdDescElement = document.createElement("td");
                tdDescElement.innerText = product.prod_desc;
                const tdPriceElement = document.createElement("td");
                tdPriceElement.innerText = product.prod_price;
                // Add elements
                trElement.appendChild(tdIdElement);
                trElement.appendChild(tdNameElement);
                trElement.appendChild(tdDescElement);
                trElement.appendChild(tdPriceElement);
                //
                tblElement.appendChild(trElement);
              });
              // Add table to div
              divElement.appendChild(tblElement);
           };
      };
    };

    // Handle form submission
    document.querySelector("form").addEventListener("submit", e => {
      // Cancel default behavior of sending a synchronous POST request
      e.preventDefault();
      // Create a FormData object, passing the form as a parameter
      const formData = new FormData(e.target);
      fetch("/searchajax", {
          method: "POST",
          body: formData
      })
          .then(response => response.json())
          .then(result => {
              displayResults(result);
          })
          .catch(err => {
              console.error(err.message);
          });
    });
    ```  
    
    Notes:  
    - Standard event listener to handle form submit via AJAX call  
    - Calling displayResults to display the table in the "output" div element
      - Created HTML Elements, assigned values, and added to HTML page.
    - Performing error checking  
    
3.  Modify index.js to add route for "get" 
    ```js
    app.get("/searchajax", async (req, res) => {
        // Omitted validation check
        const totRecs = await dblib.getTotalRecords();
        res.render("searchajax", {
            totRecs: totRecs.totRecords,
        });
    });
    ```  
    
    Notes:  
    - Very simple response.  Simply passing totRecs
    

4.  Modify index.js to add route for "post" 
    ```js
    app.post("/searchajax", upload.array(), async (req, res) => {
        dblib.findProducts(req.body)
            .then(result => res.send(result))
            .catch(err => res.send({trans: "Error", error: err.message}));

    });
    ```  
    
    Notes:  
    - Again, relatively simple response.
    - Added upload.array() as a parameter to use multer
    
# Commit to GitHub, Deploy to Azure
  Complete the following  
  - Commit changes  
  - Deploy to GitHub  
  - Deploy to Azure  
  - Validate that the application works  
  
# Conclusion  
  You should understand the difference between using a standard POST and an AJAX post.  
  Note that with AJAX there is a lot less network traffic: the entire page does not need to refresh.  You don't need to keep track of what was on the form and send it back.  The disadvantage is that you had to create the output using JavaScript.
