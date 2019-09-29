# GraphQL-101
Getting started with an Apollo GraphQL Server and NodeJS
--

# Introduction

## References

* https://www.apollographql.com/docs/apollo-server/

* * *

# Usage

## Step 1. Clone and install the dependencies

* Clone this project locally 
* Switch to the project folder
* Run:
```
npm install 
```

## Step 2. Initialize the database

This will create a file called `database.sqlite` in the project root folder.

Then it will run `sequelize db:migrate`.

```
npm run create:db
```

## Step 3. Start the Apollo Graph QL Server

```
npm start
```

## Step 4. Browse to the GraphQL Playground

Browse to: http://localhost:4000/

## Step 5. Create a user

Paste the GQL below in the left window of the playground, then press the Play button.

Note that this tutorial does not prevent duplicate user creation.

You can run this GGL several times and create several users with the same properties (only the id would change)

```js
mutation {
 createUser(
     name: "John Doe", 
     email: "john@gmail.com", 
     password: "secret") 
{
  name
  email
  id
 }
}
```

Repeat a few times, changing the name and email to create other users.

## Step 6. List all users

```js
query {
 allUsers {
  name
  email
  id
 }
}
```

## Step 7. Create a recipe

The userID should match the id returned by the command above.

```js
mutation {
  createRecipe(
    userId: 1
    title: "Honey BBQ Chicken Wings"
    ingredients: "Chicken, Honey BBQ Sauce"
    direction: "Put wings in sauce, Throw em on BBQ"
  ) {
    id
    title
    ingredients
    direction
    user {
      id
      name
      email
    }
  }
}
```

## Step 8. List all the recipes

```js
query {
 allRecipes {
	title
  direction
  ingredients
  id
  user {
    name
    email
    id
  }
 }
}
```

## Step 9. View the console

You should see database activity for the steps above that was echoed to the console.

## Step 10. Start over

To start over:

```
npm run reset:db
```

* * *

# Original project tutorial setup

Note that I've made some fixes, additions and changes.

Review the code for the latest info and examples.

## References

* https://www.apollographql.com/docs/apollo-server/
* https://scotch.io/tutorials/super-simple-graphql-with-node

# Steps

## 1. Init npm

```
npm init -y
```

## 2. Install packages

```
npm install --save sequelize sequelize-cli sqlite3
```

## 3. Initialize sequelize

```
node_modules/.bin/sequelize init
```

## 4. Change config/config.json

```js
{
  "development": {
    "dialect": "sqlite",
    "storage": "./database.sqlite"
  }
}
```

## 5. Init database file

```
touch database.sqlite
```

## 6. Create db

```
node_modules/.bin/sequelize model:create --name User --attributes name:string,email:string,password:string
```

## 7. Edit migrations/XXXXXXXXXXXXXX-create-user.js

Update the field definitions below to allowNull.

```js
name: {
    allowNull: false,
    type: Sequelize.STRING
},
email: {
    allowNull: false,
    type: Sequelize.STRING
},
password: {
    allowNull: false,
    type: Sequelize.STRING
}
```

## 8. Edit models/user.js

Update the field definitions below to allowNull.

```js
name: {
    type: DataTypes.STRING,
    allowNull: false
},
email: {
    type: DataTypes.STRING,
    allowNull: false
},
password: {
    type: DataTypes.STRING,
    allowNull: false
}
```

## 9. Create recipe model

```
node_modules/.bin/sequelize model:create --name Recipe --attributes title:string,ingredients:text,direction:text
```

## 10. Edit migrations/XXXXXXXXXXXXXX-create-recipe.js

```js
userId: {
    type: Sequelize.INTEGER.UNSIGNED,
    allowNull: false
},
title: {
    allowNull: false,
    type: Sequelize.STRING
},
ingredients: {
    allowNull: false,
    type: Sequelize.STRING
},
direction: {
    allowNull: false,
    type: Sequelize.STRING
},
```

## 11. Edit models/recipe.js

```js
title: {
    type: DataTypes.STRING,
    allowNull: false
},
ingredients: {
    type: DataTypes.STRING,
    allowNull: false
},
direction: {
    type: DataTypes.STRING,
    allowNull: false
}
```

## 12. models/user.js

Setup one-to-many relationships between our models

```js
User.associate = function (models) {
    User.hasMany(models.Recipe)
}
```

## 13. Edit models/recipe.js

Reciprocate the relationship.

```ts
Recipe.associate = function (models) {
    Recipe.belongsTo(models.User, { foreignKey: 'userId' })
}
```

## 14. Run the migrations

```
node_modules/.bin/sequelize db:migrate
```

## 15. Install graphql packages, etc

```
npm install --save apollo-server graphql bcryptjs
```

## 16. Edit src/index.js

Create a folder called src and then src/index.js:

```js
const { ApolloServer } = require('apollo-server')
const typeDefs = require('./schema')
const resolvers = require('./resolvers')
const models = require('../models')

const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: { models }
})

server
  .listen()
  .then(({ url }) => console.log('Server is running on localhost:4000'))
```

## 17. Edit src/schema.js

```js
const { gql } = require('apollo-server')

const typeDefs = gql`
    type User {
        id: Int!
        name: String!
        email: String!
        recipes: [Recipe!]!
      }

    type Recipe {
        id: Int!
        title: String!
        ingredients: String!
        direction: String!
        user: User!
    }

    type Query {
        user(id: Int!): User
        allRecipes: [Recipe!]!
        recipe(id: Int!): Recipe
    }

    type Mutation {
        createUser(name: String!, email: String!, password: String!): User!
        createRecipe(
          userId: Int!
          title: String!
          ingredients: String!
          direction: String!
        ): Recipe!
    }
`

module.exports = typeDefs
```

## 18. Edit src/resolvers.js

```js

const bcrypt = require('bcryptjs')

const resolvers = {
      Query: {
            async user(root, { id }, { models }) {
                  return models.User.findById(id)
            },
            async allRecipes(root, args, { models }) {
                  return models.Recipe.findAll()
            },
            async recipe(root, { id }, { models }) {
                  return models.Recipe.findById(id)
            }
      },
      Mutation: {
            async createUser(root, { name, email, password }, { models }) {
                  return models.User.create({
                        name,
                        email,
                        password: await bcrypt.hash(password, 10)
                  })
            },
            async createRecipe(root, { userId, title, ingredients, direction }, { models }) {
                  return models.Recipe.create({ userId, title, ingredients, direction })
            }
      },
      User: {
            async recipes(user) {
                  return user.getRecipes()
            }
      },
      Recipe: {
            async user(recipe) {
                  return recipe.getUser()
            }
      }
}

module.exports = resolvers
```

## 19. Run the server

```
node src/index.js
```

## 20. Browse GraphQL Playground 

Browse to: http://localhost:4000/

## 21. Run this mutation to create a user

This step was missing from the tutorial.

```js
mutation {
 createUser(
     name: "John Doe", 
     email: "john@gmail.com", 
     password: "secret") 
{
  name
  email
 }
}
```

## 22. Run this mutation to create a recipe

Tutorial had userId wrong. User userID: 1.

```js
mutation {
  createRecipe(
    userId: 1
    title: "Sample 2"
    ingredients: "Salt, Pepper"
    direction: "Add salt, Add pepper"
  ) {
    id
    title
    ingredients
    direction
    user {
      id
      name
      email
    }
  }
}
```

## Update package.json

```json
  "main": "src/index.js",
  "scripts": {
    "start": "node src/index.js",
    "test": "echo \"Error: no test specified\" && exit 1",
    "precreate:db": "touch database.sqlite",
    "create:db": "node_modules/.bin/sequelize db:migrate",
    "delete:db": "rm database.sqlite"
  },
```