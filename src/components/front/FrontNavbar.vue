<template>
  <nav class="navbar navbar-expand-md navbar-dark fixed-top">
    <router-link class="navbar-logo" to="/home">
      <span>
        <i class="fab fa-vuejs"></i>CP.com
      </span>
    </router-link>
    <button
      class="navbar-toggler"
      type="button"
      data-toggle="collapse"
      data-target="#navbarsExampleDefault"
      aria-controls="navbarsExampleDefault"
      aria-expanded="false"
      aria-label="Toggle navigation"
    >
      <span class="navbar-toggler-icon"></span>
    </button>

    <div class="collapse navbar-collapse" id="navbarsExampleDefault">
      <ul class="navbar-nav align-items-center mr-auto">
        <li class="nav-item">
          <router-link class="nav-link" to="/Front_check_order">CHECK ORDER</router-link>
        </li>
        <li class="nav-item">
          <router-link class="nav-link" to="/frontProducts/all">PRODUCTS</router-link>
        </li>
      </ul>

      <ul class="navbar-nav align-items-center ml-3">
        <li class="nav-item">
          <router-link class="text-primary nav-link mr-2" to="/shopping_cart/front_cart_items">
            CART
            <i class="fas fa-shopping-cart ml-1"></i>
          </router-link>
        </li>
        <li>
          <a class="text-light-danger nav-link mr-1" href="#" v-if="is_login" @click="signOut">
            SIGN OUT
            <i class="fas fa-sign-out-alt ml-1"></i>
          </a>
          <router-link class="text-secondary nav-link mr-2" to="/login" v-else>
            SIGN IN
            <i class="fas fa-sign-in-alt ml-1"></i>
          </router-link>
        </li>

        <li>
          <router-link class="text-secondary nav-link mr-2" to="/admin/products" v-if="is_login">
            BACK END
            <i class="fas fa-cog ml-1"></i>
          </router-link>
        </li>
      </ul>
    </div>
  </nav>
</template>

<script>
import $ from 'jquery';

$(function () {
  if ($(window).width() < 768) {
    $('.navbar a').on('click', function () {
      if ($('#navbarsExampleDefault').hasClass('show')) {
        $('.navbar-toggler').click();
      }
    });
  }
});

export default {
  data () {
    return {
      is_login: false
    };
  },

  methods: {
    checkLoginStatus () {
      const api = `${process.env.API_PATH}/api/user/check`;
      const vm = this;

      vm.$http.post(api).then(response => {
        vm.is_login = response.data.success;
      });
    },

    signOut () {
      const api = `${process.env.API_PATH}/logout`;
      const vm = this;
      vm.$http.post(api).then(response => {
        if (response.data.success) {
          vm.is_login = false;
          vm.$router.push('/home');
        }
      });
    }
  },

  created () {
    this.checkLoginStatus();
  }
};
</script>
