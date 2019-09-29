
## References

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

```
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

```
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

```
  "main": "src/index.js",
  "scripts": {
    "start": "node src/index.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
```