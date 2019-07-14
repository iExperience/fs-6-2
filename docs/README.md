# iXperience Full Stack 2019 - Day 6

[[toc]]


## Express

### Generate package.json

Create package.json
```bash
npm init
```

### Installation

install express
```bash
npm install express --save
```

### Main app entry point - index.js
index.js
```js
const express = require('express');

const app = express();
 
const PORT = process.env.PORT || 5000;

app.listen(PORT,()=>console.log(`Server running on port ${PORT}`));
```

### Run server locally

run server
```bash
node .
```

add run script to package.json 
```json
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start"  : "node ."
  }
```

run server
```bash
npm start
```

### Basic Route Handling - Hello World

index.js
```js
const express = require('express');

const app = express();

app.get('/', (req,res) => {
    res.send('<h1>Hello World!</h1>')
});
 
const PORT = process.env.PORT || 5000;

app.listen(PORT,()=>console.log(`Server running on port ${PORT}`));
```

### Nodemon

installing nodemon as dev dependacy
```bash
npm install -D nodemon
```

### Nodemon run scripts

adding run script to package.json
```json
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node .",
    "dev": "nodemon"
  }
```

Run server with nodemon
```bash
npm run dev
```

## Express Route Handling

### Create User routes in express

Add dumy data - index.js
```js
const users = [
    {
        "id": "1",
        "name": "Byron",
        "role": "user",
        "email": "byron@mail.com",
        "password": "P@ssword"
      },
      {
        "id": "2",
        "name": "brett",
        "role": "service provider",
        "email": "brett@mail.com",
        "password": "P@ssword1"
      },
      {
        "id": "3",
        "name": "Riaz",
        "role": "user",
        "email": "riaz@mail.com",
        "password": "P@ssword2"
      },
    ]
```

Get users route - index.js  
```js
app.get('/api/users', (req,res) => {
    res.json(users);
});
```

### Install FS and add data.json

Install fs
```bash
npm install fs
```

Add dumy data - ./src/data/data.json
```json
{
  "users":[
    {
      "id":"1",
      "name":"Byron",
      "surname":"de Villiers",
      "cellPhone":"0829123465",
      "email":"byron@mail.com",
      "password":"P@ssword",
      "role":"provider"
    }
  ]
}
```

### Update User Route - index.js

Update users route - index.js  
```js
var fs = require("fs");

router.get("/api/users", (req, res) => {
  //res.send('<h1>Hello World!</h1>');
  fs.readFile("./src/data/data.json", function(err, data) {
    if (err) throw err;
    var parseData = JSON.parse(data);
    res.json(parseData.users);
  });
});
```


### Middleware

index.js  
```js
//Middleware function:
const logger = (req,res,next) => {
    console.log(`${req.protocol}://${req.get('host')}${req.originalUrl}`);
    next();
}

//Middleware execue:
app.use(logger)
```

### Middleware

index.js  
```js
//Middleware function:
const logger = (req,res,next) => {
    console.log(`${req.protocol}://${req.get('host')}${req.originalUrl}`);
    next();
}

//Middleware execue:
app.use(logger)
```

### Get user by Id express endpoint

index.js  
```js
app.get("api/users/:id", (req, res) => {
  fs.readFile("./src/data/data.json", function(err, data) {
    if (err) throw err;
    var parseData = JSON.parse(data);
    res.json(users.filter(user => user.id === req.params.id));
  });
});
```

### Error Handling

index.js  
```js
app.get("api/users/:id", (req, res) => {
  fs.readFile("./src/data/data.json", function(err, data) {
    if (err) throw err;
    var parseData = JSON.parse(data);
    const found = parseData.users.some(user => user.id === req.params.id);
    if (found) {
      res.json(users.filter(user => user.id === req.params.id));
    } else {
      res.status(400).json({ msg: "User not found" });
    }
  });
});
```

### Express Router

./src/api/user-routes.js  
```js
const express = require("express");
const router = express.Router();
var fs = require("fs");

router.get("/", (req, res) => {
  //res.send('<h1>Hello World!</h1>');
  fs.readFile("./src/data/data.json", function(err, data) {
    if (err) throw err;
    var parseData = JSON.parse(data);
    res.json(parseData.users);
  });
});

router.get("/:id", (req, res) => {
  fs.readFile("./src/data/data.json", function(err, data) {
    if (err) throw err;
    var parseData = JSON.parse(data);
    const found = parseData.users.some(user => user.id === req.params.id);
    if (found) {
      res.json(parseData.users.filter(user => user.id === req.params.id));
    } else {
      res.status(400).json({ msg: "User not found" });
    }
  });
});

module.exports = router;
```

### Express Router - index.js

Updating express router - index.js  
```js
//Routes:
app.use("/api/users", require("./src/api/users-routes"));
```

## CRUD

### Create User

index.js
```js
//Body Parser Middlware:
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
```

user-routes.js
```js
router.post("/", (req, res) => {
  const user = req.body;
  fs.readFile("./src/data/data.json", function(err, data) {
    var error = false;
    var errMsg = "";
    if (err) {
      error = true;
      throw err;
    } else {
      var count = 0;

      if (data.length > 0) {
        var parseData = JSON.parse(data);
        parseData.users.forEach(existingUser => {
          if (existingUser.email === user.email) {
            throw new Error("This email address already been used");
          }
          count++;
        });
      } else {
        parseData = {
          users: []
        };
      }

      const newUser = {
        id: (count + 1).toString(),
        name: user.name,
        surname: user.surname,
        cellPhone: user.cellPhone,
        email: user.email,
        password: user.password,
        role: user.role
      };

      parseData.users.push(newUser);
      fs.writeFile("./src/data/data.json", JSON.stringify(parseData), function(
        err
      ) {
        if (err) {
          error = true;
          throw err;
        }
        res.json(newUser);
      });
    }

    if (error) {
      res.status(400).json({ msg: errMsg });
    } else {
      res.json(user);
    }
  });
});
```

### Update User

user-routes.js
```js
router.post("/update", (req, res) => {
  const user = req.body;
  fs.readFile("./src/data/data.json", function(err, data) {
    var error = false;
    if (err) {
      error = true;
      throw err;
    } else {
      if (data.length > 0) {
        var parseData = JSON.parse(data);
      } else {
        throw Error("No Users");
      }

      parseData.users = parseData.users.filter(existingUser => {
        return existingUser.email !== user.email;
      });

      const updateUser = {
        id: user.id,
        name: user.name,
        surname: user.surname,
        cellPhone: user.cellPhone,
        email: user.email,
        password: user.password,
        role: user.role
      };

      parseData.users.push(updateUser);
      fs.writeFile("./src/data/data.json", JSON.stringify(parseData), function(
        err
      ) {
        if (err) {
          error = true;
          throw err;
        }
        res.json(updateUser);
      });
    }

    if (error) {
      res.status(400).json({ msg: err.msg });
    } else {
      res.json(user);
    }
  });
});
```


### Delete User 

user-route.js
```js
router.get("/delete/:id", (req, res) => {
  fs.readFile("./src/data/data.json", function(err, data) {
    var error = false;
    if (err) {
      error = true;
      throw err;
    }
    var parseData = JSON.parse(data);
    parseData.users = parseData.users.filter(user => user.id === req.params.id);
    fs.writeFile("./src/data/data.json", JSON.stringify(parseData), function(
      err
    ) {
      if (err) {
        error = true;
        throw err;
      }
      res.json({ status: "User deleted" });
    });
    if (error) {
      res.status(400).json({ msg: err.message });
    } else {
      res.json({ status: "User deleted" });
    }
  });
});  
```
