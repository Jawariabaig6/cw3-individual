<template>
  <!-- Fight me -->
  <div id="app">
    <header>
      <h1 v-text="sitename"></h1>
      <!-- Search feild added -->
      <input
        type="search"
        placeholder="SearchCourse"
        v-model="SearchCourse"
        v-on:keyup="showSearch"
      />
      <button v-on:click="showCheckout">
        {{ cartItemCount }}
        <span class="fas fa-cart-plus"></span> CheckOut
      </button>

      <!-- adding radio button -->
      <div>
        <input
          type="radio"
          id="html"
          name="order"
          v-model="order"
          v-bind:value="'ascending'"
          checked
        />
        <label for="ascending">Ascending</label><br />
        <input
          type="radio"
          id="html"
          name="order"
          v-model="order"
          v-bind:value="'descending'"
        />
        <label for="descending">Descending</label><br />
      </div>
      <hr />
      <div>
        <input
          type="radio"
          id="html"
          name="sort"
          v-model="sort"
          v-bind:value="'subject'"
          checked
        />
        <label for="subject">Subject</label><br />
        <input
          type="radio"
          id="html"
          name="sort"
          v-model="sort"
          v-bind:value="'location'"
        />
        <label for="location">Location</label><br />
        <input
          type="radio"
          id="html"
          name="sort"
          v-model="sort"
          v-bind:value="'price'"
        />
        <label for="price">Price</label><br />
        <input
          type="radio"
          id="html"
          name="sort"
          v-model="sort"
          v-bind:value="'space'"
        />
        <label for="space">Available Spaces</label><br />
      </div>
    </header>
    <main>
      <div v-if="checkSearchBar">
        <div v-for="(products, index) in ProductList" :key="index">
          <!--Adding lessons component  -->
          <LessonList :products="products" :cart="cart" @addToCart="addToCart">
          </LessonList>
        </div>
      </div>
      <div v-else>
        <div v-if="ShowProduct">
          <div v-for="product in Sort" :key="product._id">
            <!-- classes information  -->
            <LessonList :products="product" :cart="cart" @addToCart="addToCart">
            </LessonList>
          </div>
        </div>
        <div v-else class="checkout">
          <h2>CheckOut Page</h2>
          <div v-for="(product,index) in cart" :key="index">
            <!-- classes information  -->
            <CheckoutList :product="product" :index="index" @remove="remove"></CheckoutList>
          </div>
          <p>
            <strong>Name: </strong>
            <input v-model.trim="name" />
          </p>
          <p>
            <strong>Phone: </strong>
            <input v-model="phone" type="number" />
          </p>
          <p>
            <!-- CheckOut -->
            <button
              v-on:click="SubmitForm"
              v-if="
                /^(?!\s*$)[a-zA-Z.+\s'-]+$/.test(this.name) &&
                /^(?!\s*$)[0-9.+\s'-]+$/.test(this.phone) &&
                this.phone.length === 10
              "
            >
              Place Order
              <i class="fas fa-shopping-cart"></i>
            </button>
            <button v-on:click="SubmitForm" disabled v-else>
              Place Order
              <i class="fas fa-shopping-cart"></i>
            </button>
          </p>
        </div>
      </div>
    </main>
  </div>
</template>

<script>
/* eslint-disable */
// components added 
import LessonList from "./components/Lesson.vue";
import CheckoutList from "./components/Checkout.vue";

export default {
  data() {
    return {
      sitename: "After School Courses",
      products: [],
      cart: [], // array to store items in cart
      temp_lesson_array: [], //temporay arry for lesson
      SearchCourse: "",
      searchInput: "",
      checkSearchBar: false,
      ShowProduct: true,
      name: "",
      phone: "",
      order: "ascending",
      sort: "subject",
    };
  },

  components: { LessonList, CheckoutList },

  // Methods start
  methods: {
    SubmitForm() {
      let uniqueArrayId = [
        ...new Map(this.cart.map((item) => [item["_id"], item])).values(),
      ];

      //Updating the number of spaces

      let store_array = [];
      uniqueArrayId.forEach((item) => {
        let temp_lesson_object = {
          id: item._id,
          name: item.subject,
          spaces_ordered: 10 - item.space,
        };

        store_array.push(temp_lesson_object);
      });
      //storing the required object

      let order_object = {
        name: this.name,
        phone: this.phone,
        ordered_lessons: store_array,
      };

      fetch("http://localhost:5000/collection/order", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(order_object),
      })
        .then((res) => res.json())
        .then((data) => {
          // do something with the response data from the POST request
          console.log({ Success: data });
        });
      // PUT start here

      store_array.forEach((item) => {
        fetch("http://localhost:5000/collection/lessons/" + item["id"], {
          method: "PUT",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ space: 10 - item["spaces_ordered"] }),
        })
          .then((res) => res.json())
          .then((data) => {
            // do something with the response data from the PUT request
            console.log({ Success: data });
          });
      });
    },
   
    addToCart(product) {
      this.cart.push(product);
      product.space--;
      console.log(this.cart);
    },
    showCheckout() {
      this.ShowProduct = this.ShowProduct ? false : true;
    },
    showSearch() {
      this.checkSearchBar = true;
    },

    // Remove funtion
    remove(id,index) {
      this.cart.splice(index, 1);

      for (let i = 0; i < this.products.length; i++) {
        if (this.products[i]["id"] === id) {
          this.products[i].space = this.products[i].space + 1;
        }
      }
    },
    // computed methods
  },
  computed: {
    cartItemCount: function () {
      return this.cart.length;
    },
    // Search fuction
    ProductList() {
      let webstore = this;
      this.checkSearchBar = true;
      if (this.SearchCourse.trim().length > 0) {
        // return this.products.filter((products) => products.subject.toLowerCase().includes(this.SearchCourse.trim().toLowerCase()) || products.location.toLowerCase().includes(this.SearchCourse.trim().toLowerCase()))
        fetch(
          "http://localhost:5000/lessons/" + this.SearchCourse.toLowerCase()
        ).then(function (response) {
          response.json().then(function (json) {
            webstore.temp_lesson_array = json;
            console.log(json);
          });
        });

        return this.temp_lesson_array;
      } else {
        this.checkSearchBar = false;
        this.ShowProduct = true;
        return this.products;
      }
    },
    // sorting the array of items
    Sort() {
      console.log("Hi");
      let arr = this.products.slice(0);
      console.log(arr);

      if (this.sort === "subject") {
        if (this.order === "ascending") {
          function compare(a, b) {
            if (a.subject > b.subject) return 1;
            if (a.subject < b.subject) return -1;
            return 0;
          }
          return arr.sort(compare);
        } else if (this.order === "descending") {
          function compare(a, b) {
            if (a.subject < b.subject) return 1;
            if (a.subject > b.subject) return -1;
            return 0;
          }
          return arr.sort(compare);
        }
      }

      if (this.sort === "location") {
        if (this.order === "ascending") {
          function compare(a, b) {
            if (a.location > b.location) return 1;
            if (a.location < b.location) return -1;
            return 0;
          }
          return arr.sort(compare);
        } else if (this.order === "descending") {
          function compare(a, b) {
            if (a.location < b.location) return 1;
            if (a.location > b.location) return -1;
            return 0;
          }
          return arr.sort(compare);
        }
      }
      if (this.sort === "price") {
        if (this.order === "ascending") {
          function compare(a, b) {
            if (a.Price > b.Price) return 1;
            if (a.Price < b.Price) return -1;
            return 0;
          }
          return arr.sort(compare);
        } else if (this.order === "descending") {
          function compare(a, b) {
            if (a.Price < b.Price) return 1;
            if (a.Price > b.Price) return -1;
            return 0;
          }
          return arr.sort(compare);
        }
      }
      if (this.sort === "space") {
        if (this.order === "ascending") {
          function compare(a, b) {
            if (a.space > b.space) return 1;
            if (a.space < b.space) return -1;
            return 0;
          }
          return arr.sort(compare);
        } else if (this.order === "descending") {
          function compare(a, b) {
            if (a.space < b.space) return 1;
            if (a.space > b.space) return -1;
            return 0;
          }
          return arr.sort(compare);
        }
      }
      console.log(arr);
    },
  },
  created: function () {
    let webstore = this;
    fetch("http://localhost:5000/collection/lessons").then(function (response) {
      response.json().then(function (json) {
        webstore.products = json;
        console.log(json);
      });
    });
  },
};
</script>

<style>
* {
  margin: 0;
  box-sizing: border-box;
  list-style-type: none;
  text-decoration: none;
  font-size: medium;
  margin-left: 11px;
}

fieldset {
  border: 3px solid #66545e;
  border-radius: 10px;
  background-color: #c0c0c0;
  width: 50%;
  padding: 10px;
  width: 190px;
  height: min-content;
  margin: auto;
  float: left;
  margin: 20px;
  padding: 9px;
  font-family: "Lucida Sans", "Lucida Sans Regular", "Lucida Grande",
    "Lucida Sans Unicode", Geneva, Verdana, sans-serif;
}

button {
  background-color: #dfdfdf;
  border-radius: 12px;
}

h2 {
  margin: 20px;
}

h1 {
  font-size: larger;
  margin: 20px;
}

header {
  text-align: center;
  font-family: "Gill Sans", "Gill Sans MT", Calibri, "Trebuchet MS", sans-serif;
}

header h1 {
  font-size: 25px;
}

.checkout {
  margin: 30px;
}

.checkout p {
  margin: 20px;
}
</style>
