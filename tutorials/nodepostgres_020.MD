# Overview  
  This tutorial builds on the previous node-postgres tutorial.  Express and EJS are added to create a web-based application.  We will create a search screen and post back records found using both AJAX and default synchronous post.  
  
  **Prerequisite**  
  Complete the previous tutorial.  
  
# Application Setup  

1.  Add the following packages
    `npm install express ejs multer`
    
2.  Create the following directories:  
    - views  
    - public  
    
3.  Modify **index.js** file.  
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
    
4.  Create the following file in the views directory  
    File name: **index.ejs**  
    Content:  
    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Asher's Node PostgreSQL Application</title>
        <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.5.3/dist/css/bootstrap.min.css" integrity="sha384-TX8t27EcRE3e/ihU7zmQxVncDAy5uIKz4rEkgIXeMed4M0jlfIDPvg6uqKI2xXr2" crossorigin="anonymous">
    </head>
    <body>
        <h1>Node PostgreSQL Application</h1>
    </body>
    </html>
    ```  
    
    Start the application and confirm that the Web Server is Running  
    
5.  Header for Navigation / Footer  
    1.  _header.ejs file (in the views directory)  
        Content:  
    ```html
    <!doctype html>
    <html lang="en">

    <head>
      <meta charset="utf-8">
      <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
      <title>Asher's Node PostgreSQL Application</title>
      <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.5.3/dist/css/bootstrap.min.css" integrity="sha384-TX8t27EcRE3e/ihU7zmQxVncDAy5uIKz4rEkgIXeMed4M0jlfIDPvg6uqKI2xXr2" crossorigin="anonymous">
    </head>

    <body>

      <div class="container">

        <nav class="navbar navbar-expand-lg navbar navbar-dark bg-dark">
          <a class="navbar-brand" href="/">Node PostgreSQL Application</a>
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
        Content:      
        ```html  
        <hr>
        <footer>
            <p>&copy; 2020 - Asher's Node PostgreSQL Application</p>
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
        
        Start / Stop the application and verify the header and footer are displayed.
        
# Search Screen (default POST)  
In this section, we create a search form for the products and display results.  
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
        - Note that `async` function and `await`.  Try the code without this.  What do you think will be returned?  
        - Did not perform validation that dblib.getTotalRecords() was successful.  You can add that and test.  
        - Added a type value "get" to the object sent to the view.  This will be used in the EJS to determine what to render based on "get" or "post" response
    
    2.  search.ejs change (modify the below single line).  Display the variable passed from index.js  
        ```html
        <h4>Total number of records in the database: <%= totRecs %></h4>
        ```  
        
        Stop / Start the application and verify that the total is displayed.  
        
4.  Create a function to return database records based on search criterion entered  
    dblib.js changes, add the following function:  
    ```js
    const findProducts = (product) => {
        // Will build query based on data provided from the form
        //  Use parameters to avoid sql injection

        // Declare variables
        var i = 1;
        params = [];
        sql = "SELECT * FROM product WHERE true";

        // Check data provided and build query as necessary
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
                    result: result.rows
                }
            })
            .catch(err => {
                return {
                    trans: "Error",
                    result: `Error: ${err.message}`
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
        //  Can get this from the page rather than using another DB call.
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
        <% if (result.result.length === 0) { %>
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
                    <% result.result.forEach(product => { %> 
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
      - Ensure that data is returned (result.result.length === 0), can chose better variable names.  
    - Display the results in a table, iterating through the results
    - Error checking is omitted but should be added.  Experiment with bad SQL and see the results.  
    
7.  Modify index.js app.get() route  
    Since we want to retrain the values the user entered in the form; we need to handle the initial request: "get"  
    There are two approaches:  
    - Send empty data from index.js  (approach shown below)
    - Check in the search.ejs view if the form is a "post" or "get".  If it is a "get", set the values to empty data.  
    
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
    <!-- Using AJAX, not form action -->
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
        // Reset output at each call
        divElement.innerHTML = "";

        if (result.trans === "Error") {
            divElement.innerHTML += `
            <h2>Application Error</h2><br>
            <p>${result.result}</p>
            `
        } else {
            if (result.result.length === 0) {
                divElement.innerHTML += `<h3>No records found!</h3>`;
            } else {
                var tdText = "";
                result.result.forEach(product => { 
                    tdText += `
                    <tr>
                        <td>${product.prod_id}</td>
                        <td>${product.prod_name}</td>
                        <td>${product.prod_desc}</td>
                        <td>${product.prod_price}</td>
                    </tr>
                 `
                });
               //divElement.innerHTML = "Test";
               divElement.innerHTML += `
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
                    ${tdText}
                </tbody>
              </table>           
            `
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
      - Used innerHtml approach.  You can also create elements, assign them values, then add to the page.
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
            .catch(err => res.send({trans: "Error", result: err.message}));

    });
    ```  
    
    Notes:  
    - Again, relatively simple response.
    - Added upload.array() as a parameter to use multer
    
# Commit to GitHub, Deploy to Heroku  
  Complete the following  
  - Commit changes  
  - Deploy to GitHub  
  - Deploy to Heroku  
  - Validate that the application works  
  
# Conclusion  
  You should be able to understand the difference between using stand POST and AJAX post.  
  Note that with AJAX there is a lot less network traffic: do not need to refresh the entire page.  You did not need to keep track of what was on the form and send it back.  The disadvantage is that you had to create the output using JavaScript.
