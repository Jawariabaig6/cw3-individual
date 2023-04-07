<template>
  <fieldset>
    <img v-bind:src="product.Image" alt="" height="100px" width="200px" />
    <h2 v-text="product.subject"></h2>
    <p v-text="product.location"></p>
    <p>Price: {{ product.Price }}</p>

    <!-- add to cart button  -->
    <button v-on:click="removeProduct(product._id,index)">
      Remove
      <i class="fas fa-shopping-cart"></i>
    </button>
  </fieldset>
</template>

<script>

export default {
    // Adding props
  name: "CheckoutList",
  props: ["product" ,"index"],
  methods: {
removeProduct (id,index) {
 
    this.$emit("remove",id,index)
}
  }
};
</script>
