<template>
  <div class="burger">
    <h3>{{ burger.name }}</h3>
    <img :src="burger.url" :alt="burger.name" />

    <ul>
      <li><span class="ingredients">{{ burger.taste }} taste</span></li>
      <li v-if="burger.gluten"><span class="ingredients">Gluten</span></li>
      <li v-if="burger.lactose"><span class="ingredients">Lactose</span></li>
    </ul>
    <button id="decreaseButton" type="button" v-on:click="decrease">-</button>
      {{ amountOrdered }}
      <button id="increaseButton" type="button" v-on:click="increase">+</button>
    

  </div>
</template>

<script>
export default {
  name: 'OneBurger',
  props: {
    burger: Object
  },
  data: function () {
    return {
      amountOrdered:0,
    };
  },
  methods: {
    decrease () {
      if (this.amountOrdered > 0) {
        this.amountOrdered--
        this.$emit('update-order', {
          burger: this.burger,
          amount: this.amountOrdered
        })
      }
    },
    increase () {
      this.amountOrdered++
      this.$emit('update-order', {
        burger: this.burger,
        amount: this.amountOrdered
      })
    }
  }
}


</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.burger{
    background-color: black;
    color: white;
    text-align: center;
   
}
.burger h2 {
    font-size: 1.5rem;
}

.burger h2, p {
    text-align: center - 0.5rem;
}

.burger img {
    height: 200px;
    width: 200px;
    display: block;
    margin-left: 2.5rem;
}

.burger ul {
    list-style-type: disc;
    list-style-position: inside;
    text-align: left;

}
.burger button {
  border-radius: 50%;
  width: 3rem;
  height: 3rem;
  font-weight: bolder;
}
#increaseButton:hover {
    background-color: green;
    cursor: pointer;
    color: white;
}
#decreaseButton:hover {
    background-color: red;
    cursor: pointer;
    color: white;
}
</style>