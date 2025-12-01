<template>
  <div class="homeViewBody">
    <header>
      <h1>Welcome to BurgerOnline</h1>
      <img
        src="/img/burgers_resturang.png"
        alt="burger resturant"
        id="headerimage"
      />
    </header>
    <main>
      <section id="burgers">
        <h2>Select burger🍔</h2>
        <p>This is where you choose your delicious burger (づ•ᴗ•)づᯓ🍔</p>

        <Burger
          v-for="burger in burgers"
          :key="burger.name"
          :burger="burger"
          v-on:update-order="updateOrder"
        />
      </section>
      <section id="contact">
        <h2>Customer information</h2>
        <p>
          This is where you provide necessary information so that we can deliver
          our great burgers to your doorstep 🍔 😋
        </p>
        <h3>Delivery information:</h3>
        <p>
          <label for="fullName">Full name</label><br />
          <input
            type="text"
            id="fullName"
            v-model="fullName"
            required="required"
            placeholder="Full name"
          />
        </p>
        <p>
          <label for="email">E-mail</label><br />
          <input
            type="email"
            id="email"
            v-model="email"
            required="required"
            placeholder="E-mail address"
          />
        </p>

        <h4>Payment method</h4>
        <select id="payment_method" v-model="paymentMethod">
          <option value="Swish">Swish</option>
          <option value="Klarna">Klarna</option>
          <option value="Card">Card</option>
          <option value="Bitcoin">Bitcoin</option>
          <option value="Cash">Cash</option>
        </select>

        <h4>Gender:</h4>
        <div class="gender-group">
          <label>
            <input type="radio" value="Man" v-model="gender" />
            Man
          </label>

          <label>
            <input type="radio" value="Woman" v-model="gender" />
            Woman
          </label>

          <label>
            <input
              type="radio"
              value="Do not wish to provide"
              v-model="gender"
            />
            Do not wish to provide
          </label>
        </div>
      </section>
      <div id="mapContainer">
        <div id="map" v-on:click="setLocation">
          Press to set delivery location
          <div
            id="customerDot"
            v-bind:style="{ left: location.x + 'px', top: location.y + 'px' }"
          >
            T
          </div>
        </div>
      </div>

      <div>
        <button id="sendButton" type="button" v-on:click="placeOrder">
          Send Delivery
          <img src="/img/sendButtondelivery.jpg" alt="delivery icon" />
        </button>
      </div>
      <div id="receipt">
        <div v-if="orderStatus" id="customerStatus">
          <p>Order status: {{ orderStatus }}</p>
        </div>
      </div>
    </main>
  </div>
  <hr />
  <footer>
    <p>Copyright &copy; 2025 BurgerOnline</p>
  </footer>
</template>

<script>
import Burger from "../components/OneBurger.vue";
import io from "socket.io-client";
import Menu from "../assets/menu.json";

function MenuItem(name, url, taste, gluten, lactose) {
  this.name = name;
  this.url = url;
  this.taste = taste;
  this.gluten = gluten;
  this.lactose = lactose;
}

const socket = io("localhost:3000");

export default {
  name: "HomeView",
  components: {
    Burger,
  },
  data: function () {
    return {
      burgers: Menu,
      fullName: "",
      email: "",
      paymentMethod: "Swish", // default value
      gender: "Man", // default value
      orderedBurgers: {},
      location: {
        x: 0,
        y: 0,
      },
      currentOrderId: null,
      orderStatus: null,
    };
  },
  created() {
    socket.on("currentQueue", (data) => {
      if (!this.currentOrderId) return;
      const myOrder = data.orders[this.currentOrderId];
      if (myOrder) {
        this.orderStatus = myOrder.status || "pending";
      }
    });
  },
  methods: {
    getOrderNumber: function () {
      return Math.floor(Math.random() * 100000);
    },
    setLocation: function (event) {
      var offset = {
        x: event.currentTarget.getBoundingClientRect().left,
        y: event.currentTarget.getBoundingClientRect().top,
      };

      this.location = {
        x: event.clientX - 30 - offset.x,
        y: event.clientY - 30 - offset.y,
      };
    },

    updateOrder({ burger, amount }) {
      if (amount > 0) {
        this.orderedBurgers[burger.name] = amount;
      } else {
        delete this.orderedBurgers[burger.name];
      }
    },
    placeOrder() {
      if (
        !this.fullName ||
        !this.email ||
        Object.keys(this.orderedBurgers).length === 0
      ) {
        alert("Please fill in all fields and select at least one burger.");
        return;
      }
      if (this.location.x === 0 && this.location.y === 0) {
        alert("Please choose a delivery location on the map.");
        return;
      }

      if (
        this.fullName ||
        this.email ||
        !Object.keys(this.orderedBurgers).length === 0 ||
        (!this.location.x === 0 && !this.location.y === 0)
      ) {
        const orderNumber = this.getOrderNumber();
        this.currentOrderId = orderNumber;

        let receipt = "Order Receipt\n";
        receipt += "Order number: " + orderNumber + "\n";
        receipt += `Name: ${this.fullName}\n`;
        receipt += `Email: ${this.email}\n`;
        receipt += `Payment Method: ${this.paymentMethod}\n`;
        receipt += `Gender: ${this.gender}\n`;

        socket.emit("addOrder", {
          orderId: orderNumber,
          details: this.location,
          orderItems: this.orderedBurgers,
          customerDetails: {
            fullName: this.fullName,
            email: this.email,
            paymentMethod: this.paymentMethod,
            gender: this.gender,
          },
        });
        alert(receipt);
      }
    },
  },
};
</script>

<style>
@import url("https://fonts.googleapis.com/css2?family=Agbalumo&family=Cormorant:wght@700&display=swap");
body {
  font-size: 2rem;
  font-family: "Times New Roman", Times, serif;
  background-color: #340a05;
}
p {
  font-size: 1em;
  padding: 0.5rem 1rem;
  line-height: 1.5;
}

h1 {
  font-family: "Agbalumo";
  font-size: 4rem;
  color: whitesmoke;
}

main {
  background-color: white;
  background: url("/img/burgerPattern.png");
  background-size: cover;
  padding-top: 1rem;
  border-radius: 2rem;
}

header {
  position: relative;
  height: 50%;
  overflow: hidden;
  margin: 0em 2em 0em 2em;
}

header img {
  width: 100%;
  height: 20rem;
  object-fit: cover;
  opacity: 0.5;
  display: block;
  border-radius: 2rem;
}

header h1 {
  position: absolute; /*positionerar texten i bilden */
  top: 50%;
  width: 100%;
  margin: 0 auto;
  text-align: center;
}

.ingredients {
  font-weight: bold;
}

#contact {
  border-color: black;
  color: white;
}
#contact h2 {
  font-size: 1.3em;
  margin: 0.5em;
  font-weight: bold;
}
#contact h3 {
  font-size: 1em;
  margin: 0.5em;
}
#contact h4 {
  font-size: 0.8em;
  margin-left: 0.5em;
  margin-top: 0.5em;
}
#contact p {
  margin: 0.5em;
  font-size: 0.5em;
}
#contact .gender-group label {
  display: block; /* one per line */
  font-size: 1rem;
  margin: 0.2rem 0;
}
#contact .gender-group label:hover {
  color: green;
  cursor: pointer;
}
#payment_method {
  margin: 1rem;
}

button:hover {
  color: greenyellow;
  cursor: pointer; /*ändrar mus*/
}

section {
  margin: 0.5em 2em;
  padding: 2rem;
  border: 2px dashed;
}

button {
  margin: 3rem;
  padding: 1rem;
  background-color: white;
}
#sendButton {
  font-size: 1rem;
  font-family: Helvetica, sans-serif;
  font-weight: bold;
  border-radius: 1.5rem;
}
#sendButton img {
  width: 2rem;
  height: auto;
  vertical-align: middle;
  margin-left: 0.5rem;
}
#sendButton:hover {
  background-image: none;
  background-color: #1a9b6a;
  color: white;
}
#sendButton:hover img {
  content: url("/img/sendButtonHover.png");
}

section div {
  margin: 1rem;
}

div {
  margin: 1rem;
}

#burgers {
  background-color: black;
  color: white;

  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(21rem, 1fr));
  gap: 1rem;
}

#burgers > h2,
#burgers > p {
  grid-column: 1 / -1; /* rubrik och text över hela bredden */
  margin-bottom: 0.2rem;
  margin-top: 0rem;
}
#burgers h2 {
  margin-left: 1rem;
}
#mapContainer {
  height: 100%;
  width: auto;
  border: 2px dashed black;
  overflow: scroll;
  font-weight: bold;
  margin: 0em 2em;
}

#map {
  position: relative;
  width: 1920px;
  height: 1078px;
  background: url("/img/polacks.jpg");
  background-size: cover;
}

#customerDot {
  position: absolute;
  background: #340a05;
  width: 30px; /* denna och under är samma som i addorder */
  height: 30px;
  color: whitesmoke;
  border-radius: 15px;
  line-height: 30px;
  text-align: center;
}
footer {
  color: white;
}
#customerStatus {
  color: white;
}
</style>
