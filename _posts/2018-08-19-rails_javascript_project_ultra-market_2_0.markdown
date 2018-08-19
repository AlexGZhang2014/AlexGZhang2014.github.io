---
layout: post
title:      "Rails + JavaScript Project: Ultra-Market 2.0"
date:       2018-08-19 21:19:48 +0000
permalink:  rails_javascript_project_ultra-market_2_0
---


For this 4th final project, we were tasked with adding client-side JavaScript functionality to our existing Ruby on Rails app, specifically using the jQuery library. JavaScript has been challenging to learn, even with some previous JS exposure prior to starting Flatiron. However, it has been quite rewarding to add jQuery to this project and see dynamic AJAX requests in action.

The first mini-goal I had for this project was to just get the JS working on each page. So instead of creating a general .js file to put my JS code in, I just added a script tag to the page where I wanted the JS functionality. An example below:
```
<script type="text/javascript" charset="utf-8">
  $(function() {
      $("#js-all-items").on("click", function(e) {
        e.preventDefault();
        $.get("/items.json", function(data) {
          for (const i in data) {
            $("#all-items").append(`
              <h4>${data[i].name}</h4>
              <p>${data[i].description}</p>
              `);
           }
        });
     });
  });
</script>
```
The above code allows the user to view a list of all items currently for sale when they click on a hyperlink. Instead of redirecting to the items index page, the list of items will populate right below the link that was clicked. jQuery is used to fetch the items with a $.get request. Once the data (which is an array of JSON objects) is returned, the appropriate data is added to a div element with the id "all-items" using an ES6 for loop.

Once I had the jQuery functions working using script tags, I transitioned to moving all the code into the appropriate .js file, which was item.js. After I had all my code in one place, I set to work on refactoring the code such that all JSON responses would become JS Model Objects. The code below demonstrates how this is done using pre-ES6 syntax:
```
function Item(attributes) {
  this.name = attributes.name;
  this.description = attributes.description;
  this.id = attributes.id;
  this.bought = attributes.bought;
  this.merchant = attributes.merchant;
}

Item.submitCreateForm = function(e) {
  e.preventDefault();
  let $form_values = $(this).serialize();
  $.post("/items", $form_values, Item.successCreate, "json");
}

Item.listClick = function(e) {
  e.preventDefault();
  $.get("/items.json", Item.successList);
}

Item.nextItem = function() {
  let nextId = parseInt($("#js-next-item").attr("data-id")) + 1;
  $.get("/items/" + nextId + ".json", Item.successNextItem);
}

Item.successCreate = function(data) {
  let item = new Item(data);
  return item.createItem();
}

Item.successList = function(data) {
  for (const i in data) {
  let item = new Item(data[i]);
  item.listItem();
  }
}

Item.successNextItem = function(data) {
  let item = new Item(data);
  return item.displayNextItemData();
}

Item.prototype.createItem = function() {
  $("#itemName").text(this.name);
  $("#itemDescription").text(this.description);
  $("#itemMerchant").text(`Sold by: ${this.merchant.name}`);
}

Item.prototype.listItem = function() {
  $("#all-items").append(`
  <h4>${this.name}</h4>
  <p>${this.description}</p>
  `);
}

Item.prototype.displayNextItemData = function() {
  $("#itemName").text(this.name);
  $("#itemDescription").text(this.description);
  $("#itemMerchant").text(this.merchant.name);
  $("#js-next-item").attr("data-id", this.id);
}

Item.addJavaScriptListener = function() {
  $("form#new_item").submit(Item.submitCreateForm);
  $("#js-all-items").on("click", Item.listClick);
  $("#js-next-item").on("click", Item.nextItem);
}

Item.ready = function() {
  Item.addJavaScriptListener();
}

$(function() {
  Item.ready();
});
```
The above code demonstrates that all the jQuery functionality regarding AJAX requests are encapsulated within an Item object. Any method that is used by a specific instance of an Item object is defined on the Item prototype. And all of the code is encapsulated in the $(document).ready() function defined at the very bottom. When the page is loaded, Item.ready() fires, which invokes the Item.addJavaScriptListener() method, which adds event listeners to 3 elements on the page. When one of those elements is clicked (or submitted), then the appropriate callback function is invoked, which leads to more function chaining until the final result is displayed on the page.

Once I had everything working for my Item class, I modified it so that it could work for my Service class as well. Then I made a merchant.js file to allow for one specific piece of functionality that concerned the Merchant class. Finally, I decided to refactor all of my JS code to be adherent to ES6 syntax - that is, I converted each Object prototype into a Class. See the example below:
```
class Item {
  constructor(attributes) {
    this.name = attributes.name;
    this.description = attributes.description;
    this.id = attributes.id;
    this.bought = attributes.bought;
    this.merchant = attributes.merchant;
  }

  static submitCreateForm(e) {
    e.preventDefault();
    let $form_values = $(this).serialize();
    $.post("/items", $form_values, Item.successCreate, "json");
  }

  static listClick(e) {
    e.preventDefault();
    $.get("/items.json", Item.successList);
  }

  static nextItem() {
    let nextId = parseInt($("#js-next-item").attr("data-id")) + 1;
    $.get("/items/" + nextId + ".json", Item.successNextItem);
  }

  static successCreate(data) {
    let item = new Item(data);
    return item.createItem();
  }

  static successList(data) {
    for (const i in data) {
      let item = new Item(data[i]);
      item.listItem();
    }
  }

  static successNextItem(data) {
    let item = new Item(data);
    return item.displayNextItemData();
  }

  createItem() {
    $("#itemName").text(this.name);
    $("#itemDescription").text(this.description);
    $("#itemMerchant").text(`Sold by: ${this.merchant.name}`);
  }

  listItem() {
    $("#all-items").append(`
      <h4><a href="/items/${this.id}">${this.name}</a></h4>
      <p>${this.description}</p>
      `);
  }

  displayNextItemData() {
    $("#itemName").text(this.name);
    $("#itemDescription").text(this.description);
    $("#itemMerchant").text(this.merchant.name);
    $("#js-next-item").attr("data-id", this.id);
  }

  static addJavaScriptListener() {
    $("form#new_item").submit(Item.submitCreateForm);
    $("#js-all-items").on("click", Item.listClick);
    $("#js-next-item").on("click", Item.nextItem);
  }

  static ready() {
    Item.addJavaScriptListener();
  }
}

$(function() {
  Item.ready();
});
```
All of the core functions and callbacks are the same, but everything is now in ES6 syntax. Everything is encapsulated within the Item Class, which uses a constructor function when a new instance of an Item is created. All methods that were originally defined on Item.prototype are now just directly defined in the class. All class methods (such as Item.ready()) are now defined using the static keyword. Once I determined that everything still worked for the Item class, I modified my code for Service and Merchant objects to also use the Class pattern.

This project did not take as long as building the initial rails app, but it was not any less challenging. I'm satisfied with the outcome, but I know that Ultra-Market can always be improved upon, so I can't wait for feedback after my technical assessment. Coming up next...React and Redux!
