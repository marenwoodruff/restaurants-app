# Restaurants App

App created with express generator + sequelize

## App Setup

### Create the express app with necessary npm modules

- `express yum-express-sequelize-lab --git --view=ejs`
- `cd yum-express-sequelize-lab`
- `npm i`
- `npm i --save body-parser ejs-layouts method-override`
- `npm i sequelize pg pg-hstore --save`

### Create your postgres database
- `createdb yum-app-development`

### Add sequelize and configure your config.json

- `sequelize init`
- update `config.json`

### Add npm start script that uses nodemon
- update package.json to include `"start:dev": "nodemon ./bin/www`

## Restaurants Model + Controller

### Create your restaurant model + run migration

- `sequelize model:generate --name restaurant --attributes name:string,city:string,state:string,zipCode:string,typeOfFood:string,rating:integer,yelpUrl:string`
- `sequelize db:migrate`

### Create a seeds file for your restaurant

- `sequelize seed:generate --name restaurant-seeds`
- add seeds for 2 restaurants
```
module.exports = {
  up: (queryInterface, Sequelize) => {
    /*
      Add altering commands here.
      Return a promise to correctly handle asynchronicity.

      Example:
      return queryInterface.bulkInsert('Person', [{
        name: 'John Doe',
        isBetaMember: false
      }], {});
    */
    return queryInterface.bulkInsert('restaurants', [
      {
        name: 'Screwtop',
        city: 'Clarendon',
        state: 'Virginia',
        zipCode: '22201',
        typeOfFood: 'wine + American',
        rating: 4,
        yelpUrl: 'https://www.yelp.com/biz/screwtop-wine-bar-arlington-2',
        createdAt: Sequelize.literal('NOW()'),
        updatedAt: Sequelize.literal('NOW()')
      },
      {
        name: 'Barcelona',
        city: 'Washington',
        state: 'DC',
        zipCode: '20005',
        typeOfFood: 'wine + tapas',
        rating: 4,
        yelpUrl: 'https://www.yelp.com/biz/barcelona-wine-bar-washington',
        createdAt: Sequelize.literal('NOW()'),
        updatedAt: Sequelize.literal('NOW()')
      }
    ], {});
  },

  down: (queryInterface, Sequelize) => {
    /*
      Add reverting commands here.
      Return a promise to correctly handle asynchronicity.

      Example:
      return queryInterface.bulkDelete('Person', null, {});
    */
    return queryInterface.bulkDelete('restaurants', null, {});
  }
};
```
- `sequelize db:seed:all`
- check your database
  - psql
  - \l
  - \c yum-app-development
  - \dt
  - \d restaurants
  - SELECT * FROM restaurants;
  
### Create a restaurants controller

- `touch routes/restaurants_controller.js`
- add in 
  - `const express = require('express');`
  - `const router = express.Router();`
  - `const Restaurant = require('../models').restaurant;`
  - `module.exports = router;`
- require controller in app.js
  - `const restaurantsRouter = require('./routes/restaurants_controller');`
  - `app.use('/restaurants', restaurantsRouter);`

### Create index route

```
router.get('/', (req, res) => {
  Restaurant.findAll()
    .then((restaurants) => {
      res.json(restaurants);
    })
    .catch((err) => {
      res.json(err);
    });
});
```

- `mkdir views/restaurants`
- `touch views/restaurants/index.ejs`
- check that this works in the Postman

### Set up express-ejs-layous

- In your app.js, add:
  - `const ejsLayouts = require('express-ejs-layouts');`
  - `app.use('/restaurants', restaurantsRouter);`
- Create a `layout.ejs` file, and add in boilerplate html
- Dry up the `index.ejs` file

### Create index view

- Update controller route

```
router.get('/', (req, res) => {
  Restaurant.findAll()
    .then((restaurants) => {
      res.render('restaurants/index', {
        restaurants
      });
    })
    .catch((err) => {
      console.log(err);
    });
});
```

- Create `views/restaurants` folder
- Create `views/restaurants/index.ejs`

```
<h1 class="margin-bottom">Restaurants</h1>

<% if (restaurants.length) { %>

  <ul class="margin-bottom">
    <% restaurants.forEach((restaurant) => { %>
      <li class="margin-bottom">
        <h3><a href="/restaurants/<%= restaurant.id %>"><%= restaurant.name %></a></h3>
        <p><%= restaurant.typeOfFood %></p>
        <p><%= restaurant.city %>, <%= restaurant.state %> <%= restaurant.zipCode %></p>
        <p>rating: <%= restaurant.rating %></p>
        <p><a href="<%= restaurant.yelpUrl %>" target="_blank">Yelp Link</a></p>
      </li>
    <% }); %>
  </ul>

  <a href="/restaurants/new" class="btn btn-outline-dark">New Restaurant</a>

<% } else { %>
  <h3 class="margin-bottom">Add a new restaurant!</h3>
  <a href="/restaurants/new" class="btn btn-outline-dark">New Restaurant</a>
<% } %>
```

### Create the show route

- Create a controller route

```
router.get('/:id', (req, res) => {
  Restaurant.findById(req.params.id)
    .then((restaurant) => {
      res.json(restaurant);
    })
    .catch((err) => {
      console.log(err);
    });
});
```

- check in postman
- git commit

### Create a show.ejs

- Touch a `views/restaurants/show.ejs`

```
<h1><%= restaurant.name %></h1>
<h3><%= restaurant.typeOfFood %></h3>
<p><%= restaurant.city %>, <%= restaurant.state %> <%= restaurant.zipCode %></p>
<p>rating: <%= restaurant.rating %></p>
<p class="margin-bottom"><a href="<%= restaurant.yelpUrl %>" target="_blank">yelp</a></p>

<a href="/restaurants" class="btn btn-outline-dark">home</a>
```

- update the controller route

```
router.get('/:id', (req, res) => {
  Restaurant.findById(req.params.id)
    .then((restaurant) => {
      res.render('restaurants/show', {
        restaurant
      });
    })
    .catch((err) => {
      console.log(err);
    });
});
```

- check in the browser
- add a link to the homepage to the show page

### Create a new route

```
router.get('/new', (req, res) => {
  res.render('restaurants/new');
});
```

### Create a new view

- Touch a views/restaurants/new.ejs

```
<h1>add a new restaurant</h1>

<form action="/restaurants" method="POST" class="col-md-6 offset-md-3 margin-bottom">
  <div class="form-group">
    <label for="name">restaurant name</label>
    <input type="text" class="form-control" id="name" name="name">
  </div>
  <div class="form-group">
    <label for="city">city</label>
    <input type="text" class="form-control" id="city" name="city">
  </div>
  <div class="form-group">
    <label for="state">state</label>
    <input type="text" class="form-control" id="state" name="state">
  </div>
  <div class="form-group">
    <label for="zipCode">zip code</label>
    <input type="text" class="form-control" id="zipCode" name="zipCode">
  </div>
  <div class="form-group">
    <label for="typeOfFood">type of food</label>
    <input type="text" class="form-control" id="typeOfFood" name="typeOfFood">
  </div>
  <div class="form-group">
    <label for="rating">rating</label>
    <input type="number" class="form-control" id="rating" name="rating" min="0" max="5" value="0">
  </div>
  <div class="form-group">
    <label for="yelpUrl">yelp url</label>
    <input type="text" class="form-control" id="yelpUrl" name="yelpUrl">
  </div>
  <button type="submit" class="btn btn-outline-dark">submit</button>
</form>

<a href="/restaurants" class="btn btn-outline-dark">home</a>
```

### Create a post route

```
router.post('/', (req, res) => {
  Restaurant.create(req.body)
    .then((restaurant) => {
      res.redirect(`/restaurants/${restaurant.id}`);
    })
    .catch((err) => {
      console.log(err);
    });
});
```

### Add an edit route

```
router.get('/:id/edit', (req, res) => {
  Restaurant.findById(req.params.id)
    .then((restaurant) => {
      res.render('restaurants/edit', {
        restaurant
      });
    })
    .catch((err) => {
      console.log(err);
    });
});
```

### Create an views/restaurants/edit.ejs

```
<h1>edit <%= restaurant.name %> restaurant</h1>

<form action="/restaurants/<%= restaurant.id %>?_method=PUT" method="POST" class="col-md-6 offset-md-3 margin-bottom">
  <div class="form-group">
    <label for="name">restaurant name</label>
    <input type="text" class="form-control" id="name" name="name" value="<%= restaurant.name %>">
  </div>
  <div class="form-group">
    <label for="city">city</label>
    <input type="text" class="form-control" id="city" name="city" value="<%= restaurant.city %>">
  </div>
  <div class="form-group">
    <label for="state">state</label>
    <input type="text" class="form-control" id="state" name="state" value="<%= restaurant.state %>">
  </div>
  <div class="form-group">
    <label for="zipCode">zip code</label>
    <input type="text" class="form-control" id="zipCode" name="zipCode" value="<%= restaurant.zipCode %>">
  </div>
  <div class="form-group">
    <label for="typeOfFood">type of food</label>
    <input type="text" class="form-control" id="typeOfFood" name="typeOfFood" value="<%= restaurant.typeOfFood %>">
  </div>
  <div class="form-group">
    <label for="rating">rating</label>
    <input type="number" class="form-control" id="rating" name="rating" min="0" max="5" value="<%= restaurant.rating %>">
  </div>
  <div class="form-group">
    <label for="yelpUrl">yelp url</label>
    <input type="text" class="form-control" id="yelpUrl" name="yelpUrl" value="<%= restaurant.yelpUrl %>">
  </div>
  <button type="submit" class="btn btn-outline-dark">submit</button>
</form>

<a href="/restaurants" class="btn btn-outline-dark">home</a>
```

### Create a put route

```
router.put('/:id', (req, res) => {
  Restaurant.findById(req.params.id)
    .then((restaurant) => {
      return restaurant.update(req.body);
    })
    .then((updatedRestaurant) => {
      res.render('restaurants/show', {
        restaurant: updatedRestaurant
      });
    })
    .catch((err) => {
      console.log(err);
    });
});
```

### Create a delete route

```
router.delete('/:id', (req, res) => {
  Restaurant.findById(req.params.id)
    .then((restaurant) => {
      return restaurant.destroy();
    })
    .then(() => {
      res.redirect('/restaurants');
    })
    .catch((err) => {
      console.log(err);
    });
});
```

### Create a delete form

```
<form action="/restaurants/<%= restaurant.id %>?_method=DELETE" method="POST" class="margin-bottom">
  <input type="submit" value="delete restaurant" class="btn btn-outline-danger">
</form>
```

## Menu Model + Controller

### Create your menu model + run migration

- `sequelize model:generate --name menu --attributes name:string,days:string,hours:string,restaurantId:integer`
- update associations
- `sequelize db:migrate`

### Create a seeds file for your menu

- `sequelize seed:generate --name menu-seeds`
- add seeds for 2 menus

```
module.exports = {
  up: (queryInterface, Sequelize) => {
    /*
      Add altering commands here.
      Return a promise to correctly handle asynchronicity.

      Example:
      return queryInterface.bulkInsert('Person', [{
        name: 'John Doe',
        isBetaMember: false
      }], {});
    */
    return queryInterface.bulkInsert('restaurants', [
      {
        name: 'Brunch',
        days: 'Saturday + Sunday',
        hours: '11:00-3:00',
        restaurantId: 1,
        createdAt: Sequelize.literal('NOW()'),
        updatedAt: Sequelize.literal('NOW()')
      },
      {
        name: 'Brunch',
        days: 'Saturday + Sunday',
        hours: '11:00-3:00',
        restaurantId: 2,
        createdAt: Sequelize.literal('NOW()'),
        updatedAt: Sequelize.literal('NOW()')
      },
    ], {});
  },

  down: (queryInterface, Sequelize) => {
    /*
      Add reverting commands here.
      Return a promise to correctly handle asynchronicity.

      Example:
      return queryInterface.bulkDelete('Person', null, {});
    */
    return queryInterface.bulkDelete('restaurants', null, {});
  }
};

- run `sequelize db:seed:all`
- check your database
  - psql
  - \l
  - \c yum-app-development
  - \dt
  - \d restaurants
  - SELECT * FROM restaurants;

### Create a menus_controller.js

- `touch routes/menus_controller.js`
- add in 
  - `const express = require('express');`
  - `const router = express.Router();`
  - `const Menu = require('../models').menu;`
  - `module.exports = router;`
- require controller in app.js
  - `const menusRouter = require('./routes/menus_controller');`
  - `app.use('/restaurants', menusRouter);`

### Add Menu model to Restaurants controller

- `const Menu = require('../models').menu;`

```
router.get('/:id', (req, res) => {
  Restaurant.findById(req.params.id, {
    include: [ Menu ]
  })
    .then((restaurant) => {
      res.render('restaurants/show', {
        restaurant
      });
    })
    .catch((err) => {
      console.log(err);
    });
});
```

### Update Restaurants show page

```
<h1><%= restaurant.name %></h1>
<h3><%= restaurant.typeOfFood %></h3>
<p><%= restaurant.city %>, <%= restaurant.state %> <%= restaurant.zipCode %></p>
<p>rating: <%= restaurant.rating %></p>
<p class="sm-margin-bottom"><a href="<%= restaurant.yelpUrl %>" target="_blank">yelp</a></p>

<h3>Menus</h3>

<% if (restaurant.menus.length) { %>
  <% restaurant.menus.forEach((menu) => { %>
    <p class="margin-bottom"><a href="/restaurants/<%= restaurant.id %>/menus/<%= menu.id %>"><%= menu.name %></a></p>
  <% }) %>
<% } else { %>
  <a href="/restaurants/<%= restaurant.id %>/menus/new">add a new menu</a>
<% } %>

<a href="/restaurants/<%= restaurant.id %>/edit" class="btn btn-outline-dark sm-margin-bottom">edit restaurant</a>

<form action="/restaurants/<%= restaurant.id %>?_method=DELETE" method="POST" class="margin-bottom">
  <input type="submit" value="delete restaurant" class="btn btn-outline-danger">
</form>

<a href="/restaurants" class="btn btn-outline-dark">home</a>
```

### Add a Menu show route

```
router.get('/:restaurantId/menus/:id', (req, res) => {
  Menu.findById(req.params.id)
    .then((menu) => {
      let restaurantId = menu.restaurantId;
      res.render('menus/show', {
        restaurantId,
        menu
      })
    })
    .catch((err) => {
      console.log(err);
    });
});
```

### Create a menu show page

- `mkdir views/menus`
- `touch views/menus/show.ejs`

```
<h1><%= menu.name %></h1>
<h3>days: <%= menu.days %></h3>
<p class="margin-bottom">hours: <%= menu.hours %></p>


<a href="/restaurants/<%= restaurantId %>/menus/edit">edit menu</a>
<a href="/restaurants/<%= restaurantId %>" class="btn btn-outline-dark">back</a>
```

### Create a new menu route

```
router.get('/:restaurantId/menus/new', (req, res) => {
  Restaurant.findById(req.params.restaurantId)
    .then((restaurant) => {
      res.render('menus/new', {
        restaurant
      });
    })
    .catch((err) => {
      console.log(err);
    });
});
```

### Create a new menu view

```
<h1>add a new menu for <%= restaurant.name %></h1>

<form action="/restaurants/<%= restaurant.id %>" method="POST" class="col-md-6 offset-md-3 margin-bottom">
  <div class="form-group">
    <label for="name">name</label>
    <input type="text" class="form-control" id="name" name="name">
  </div>
  <div class="form-group">
    <label for="days">days</label>
    <input type="text" class="form-control" id="days" name="days">
  </div>
  <div class="form-group">
    <label for="hours">hours</label>
    <input type="text" class="form-control" id="hours" name="hours">
  </div>
  <button type="submit" class="btn btn-outline-dark">submit</button>
</form>

<a href="/restaurants/<%= restaurant.id %>" class="btn btn-outline-dark">back</a>
<a href="/restaurants" class="btn btn-outline-dark">home</a>
```

### Create a post route

```
router.post('/:restaurantId/menus', (req, res) => {
  let restaurantId = req.params.restaurantId;
  let menu = Menu.build(req.body);
  menu.restaurantId = restaurantId;

  menu.save()
    .then((savedMenu) => {
      res.redirect(`/restaurants/${restaurantId}`);
    })
    .catch((err) => {
      console.log(err);
    });
});
```

### Create an edit route

```
router.get('/:restaurantId/menus/:id/edit', (req, res) => {
  let menuId = req.params.id;
  Menu.findById(menuId)
    .then((menu) => {
      res.render('menus/edit', {
        menu
      });
    })
    .catch((err) => {
      console.log(err);
    });
});
```

### Create an edit view

```
<h1>update <%= menu.name %></h1>

<form action="/restaurants/<%= menu.restaurantId %>/menus/<%= menu.id %>?_method=PUT"
  method="POST"
  class="col-md-6 offset-md-3 margin-bottom">
  <div class="form-group">
    <label for="name">name</label>
    <input type="text" class="form-control" id="name" name="name" value="<%= menu.name %>">
  </div>
  <div class="form-group">
    <label for="days">days</label>
    <input type="text" class="form-control" id="days" name="days" value="<%= menu.days %>">
  </div>
  <div class="form-group">
    <label for="hours">hours</label>
    <input type="text" class="form-control" id="hours" name="hours" value="<%= menu.hours %>">
  </div>
  <button type="submit" class="btn btn-outline-dark">submit</button>
</form>

<a href="/restaurants/<%= menu.restaurantId %>/menus/<%= menu.id %>" class="btn btn-outline-dark">back</a>
<a href="/restaurants" class="btn btn-outline-dark">home</a>
```

### Create an update route

```
router.put('/:restaurantId/menus/:id', (req, res) => {
  let restaurantId = req.params.restaurantId;
  let menuId = req.params.id;
  Menu.findById(menuId, {
    include: [ Restaurant ]
  })
    .then((menu) => {
      return menu.update(req.body);
    })
    .then((updatedMenu) => {
      res.redirect(`/restaurants/${restaurantId}/menus/${menuId}`);
    })
    .catch((err) => {
      console.log(err);
    });
});
```

### Create a delete route

```
router.delete('/:restaurantId/menus/:id', (req, res) => {
  let restaurantId = req.params.restaurantId;
  let menuId = req.params.id;
  Menu.findById(menuId)
    .then((menu) => {
      menu.destroy();
    })
    .then(() => {
      res.redirect(`/restaurants/${restaurantId}`);
    })
    .catch((err) => {
      console.log(err);
    });
});
```

### Create a delete form

```
<form action="/restaurants/<%= restaurantId %>/menus/<%= menu.id %>?_method=DELETE"
  method="POST">
  <input type="submit" class="btn btn-outline-danger margin-bottom" value="delete menu">
</form>
```
