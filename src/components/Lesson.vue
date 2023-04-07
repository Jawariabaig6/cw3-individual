<template>
    <fieldset>
        <img v-bind:src="products.Image" alt="" height="100px" width="200px" />
        <h2 v-text="products.subject"></h2>
        <p v-text="products.location"></p>
        <p>Price: {{ products.Price }}</p>
        <p>Space:{{ products.space }}</p>
        <!-- add to cart button  -->
        <button v-on:click="addproduct(products)" v-if="canAddToCart(products)">
            Add to cart
            <i class="fas fa-shopping-cart"></i>
        </button>
        <button disabled="disabled" v-else>Add to cart</button>
    </fieldset>
</template>

<script>

export default {
    name: "LessonList",
    props:
    // Adding props
     [
        "products",
        "cart"
    ],
    methods:{
        canAddToCart(product) {
      return product.space > this.cartCount(product._id);
    },
    cartCount(id) {
      let count = 0;
      for (let i = 0; i < this.cart.length; i++) {
        if (this.cart[i] === id) {
          count++;
        }
      }
      return count;
    },
    addproduct(product){
        this.$emit("addToCart",product)
    }
    }
}

</script>


