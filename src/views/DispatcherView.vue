<template>
  <div id="orders">
    <div id="orderList">
      <div
        v-for="(order, key) in orders"
        :key="'order' + key"
        v-on:mouseover="hoveredOrder = key"
        v-on:mouseleave="hoveredOrder = null"
      >
        <div>#{{ key }}</div>
        <div id="customerDetails">
          <p>Name: {{ order.customerDetails.fullName }}</p>
          <p>Email: {{ order.customerDetails.email }}</p>
          <p>Payment: {{ order.customerDetails.paymentMethod }}</p>
          <p>Gender: {{ order.customerDetails.gender }}</p>
        </div>
        <ul>
          <li v-for="(amount, name) in order.orderItems" :key="name">
            {{ name }}: {{ amount }}
          </li>
        </ul>
        <p id="statusText">Status: {{ order.status || "Pending" }}</p>
        <button
          id="InPreperation"
          v-on:click="changeStatus(key, 'In preperation')"
        >
          In preperation
        </button>
        <button id="Done" v-on:click="changeStatus(key, 'Done')">Done</button>
      </div>
      <button v-on:click="clearQueue">Clear Queue</button>
    </div>
    <div id="dots">
      <div
        v-for="(order, key) in orders"
        :key="'dots' + key"
        :style="{ left: order.details.x + 'px', top: order.details.y + 'px' }"
        :class="{ highlightedDot: hoveredOrder === key }"
      >
        {{ key }}
      </div>
    </div>
  </div>
</template>
<script>
import io from "socket.io-client";
const socket = io("localhost:3000");

export default {
  name: "DispatcherView",
  data: function () {
    return {
      orders: null,
      hoveredOrder: null,
    };
  },
  created: function () {
    socket.on("currentQueue", (data) => (this.orders = data.orders));
  },
  methods: {
    clearQueue: function () {
      socket.emit("clearQueue");
    },
    changeStatus: function (orderId, status) {
      socket.emit("changeStatus", { orderId: orderId, status: status });
    },
  },
};
</script>
<style>
#orderList {
  top: 1em;
  left: 1em;
  position: absolute;
  z-index: 2;
  color: black;
  background: rgba(255, 255, 255, 0.5);
  padding: 1em;
}
#dots {
  position: relative;
  margin: 0;
  padding: 0;
  background-repeat: no-repeat;
  width: 1920px;
  height: 1078px;
  cursor: crosshair;
  background-image: url("/img/polacks.jpg");
}

#dots div {
  position: absolute;
  background: black;
  color: white;
  border-radius: 10px;
  width: 20px;
  height: 20px;
  text-align: center;
}
#customerDetails p {
  font-size: 1rem;
  padding: 0rem;
  margin: 0rem;
}
#orderList ul {
  font-size: 1rem;
}
#statusText {
  font-size: 1rem;
  margin: 0rem;
  font-weight: bold;
  color: green;
}
#InPreperation {
  border-radius: 1rem;
  margin-top: 0rem;
  margin-bottom: 0.5rem;
  display: flex;
}

#Done {
  border-radius: 1rem;
  margin-top: 0rem;
  display: flex;
}
.highlightedDot {
  transform: scale(2);
  transition: 0.2s ease;
  background-color: green !important;
}
</style>
