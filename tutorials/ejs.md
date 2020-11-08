# Overview
This tutorial provides step-by-step instructions for setting up a simple web site using [EJS](https://ejs.co/) in a Node project.  

Assumptions:
- Using VSCode
- Using Express Framework

# What is EJS?
[EJS](https://ejs.co/) is a simple templating language that lets you generate HTML markup with plain JavaScript.  
Reference
- [EJS Site](https://ejs.co/)
- [EJS GitHub](https://github.com/mde/ejs)

# Steps

## File/Folder Structure
At completion of the project, your file/folder structure will be as follows
```
  - css
    -- styles.css
  - node_modules
  - views
    -- _footer.ejs
    -- _header.ejs
    -- formAjax.ejs
    -- formPost.ejs
    -- services.ejs
    -- welcome.ejs
  - index.js
  - package-lock.json
  - package.json
```

## 1. Create a Node Project with Express and EJS modules
1. Launch VSCode
2. Create a folder for the project.  
   - Folder name: **tutejs**  
3. Open **tutejs** folder in Integrated Terminal
4. Initiate a Node project  
   ``` npm init ``` and answer the questions  
   or  
   ``` npm init -y ``` to accept defaults
   
   - Add the "start" script to the generated package.json file:  
     ``` "start": "node index.js", ```  
     package.json contents (similar, depending on your answers)
     ```js
     {
      "name": "tutejs",
      "version": "1.0.0",
      "description": "EJS Tutorial",
      "main": "index.js",
      "scripts": {
        "start": "node index.js",
        "test": "echo \"Error: no test specified\" && exit 1"
      },
      "author": "Asher",
      "license": "ISC"
     }
     ```  
 5. Intall Express and EJS modules  
    ``` npm install express ejs ```
    
## 2. Create EJS Views and Stylesheet
### EJS
We will create a header and footer views to be included in our other pages.  
1. _header.ejs content:
```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>EJS Tutorial</title>
    <link href="/styles.css" rel="stylesheet" type="text/css">
</head>
<body>
    <table width="100%">
        <tr>
            <td>EJS Tutorial: <a href="/">Welcome</a> |
                <a href="/services">Services</a> |
                <a href="/formPost">Form Post</a> |
                <a href="/formAjax">Form Ajax</a> 
            </td>
        </tr>
    </table>
```  
2. _footer.ejs content:
```js
    <hr>
    <footer>
        &copy 2020 Internet Application Development
    </footer>
</body>
</html>
```  
3. The welcome page will be our initial start page.  
welcome.ejs content:
```js
<%- include("_header") -%>

<h1>Welcome to our site</h1>

<%- include("_footer") -%>
```  
### Stylesheet
For now, we will create a simple style sheet.  
In a later tutorial, we will use [bootstrap](https://getbootstrap.com/).  
1. Create a folder for the stylesheet
   - Folder name: **css**  
2. Create a file 
   - File name: styles.css
   content:
   ```css
   * {
    font-family: Calibri;
   }

   table {
    background-color:  skyblue;
   }
   ```

## 3. Setup The Server  


