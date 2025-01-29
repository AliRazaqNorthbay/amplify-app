<template>
  <div>
    <div id="app">
      <div class="app-header">
        <div class="app-logo">
          <img src="https://aws-amplify.github.io/images/Logos/Amplify-Logo-White.svg" alt="AWS Amplify"/>
        </div>
        <h1>Welcome to the amplify framework</h1>
        <h2>Welcome to the amplify 45</h2>
      </div>
      <amplify-authenticator>
        <div class="welcome">
          <h1>Hey, {{ user.username }}!</h1>
          <amplify-sign-out></amplify-sign-out>
        </div>
      </amplify-authenticator>
    </div>
    <div class="form-body">
      <form v-on:submit.prevent autocomplete="off">
        <div>
          <label>Name: </label>
          <input v-model='form.name' class='input' autocomplete="off"/>
        </div>
        <div>
          <label>Description: </label>
          <input v-model='form.description' class='input' autocomplete="off"/>
        </div>
        <div>
          <label>City: </label>
          <input v-model='form.city' class='input' autocomplete="off"/>
        </div>
        <button @click='createRestaurant' class='button'>Create</button>
        <button @click='getData' class='button'>Create</button>
      </form>
    </div>
    <div class="app-body">
      <div v-if="loading" class="loading">Loading ....</div>
      <div class="card-container" v-if="!loading">
        <div class="card" v-for="restaurant of sortedRestaurants" :key="restaurant.id">
          <div class="remove"><button @click='deleteRestaurant(restaurant)' class='button'>Delete</button></div>
          <div class="name">{{ restaurant.city }}</div>
          <div class="price">{{ restaurant.name }}</div>
          <div class="symbol">{{ restaurant.description }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

import { AuthState, onAuthUIStateChange } from "@aws-amplify/ui-components";
import { API, graphqlOperation } from 'aws-amplify';
import{ listRestaurants } from './graphql/queries';
import { createRestaurant} from './graphql/mutations';
import { onCreateRestaurant } from './graphql/subscriptions';

export default {
  name: 'App',
  data(){
    return{
      user: {},
      restaurants: [],
      form: {},
      loading: true
    };
  },
  computed:{
    sortedRestaurants(){
      let restaurants = [...this.restaurants];
      return restaurants.sort((a, b) => a.name.localeCompare(b.name));
    }
  },
  created(){
    onAuthUIStateChange((state, user) => {
      if(state == AuthState.SignedIn){
        this.user = user;
        this.getData();
      }
    });
    API.graphql(graphqlOperation(onCreateRestaurant))
    .subscribe((sourceData) => {
      const newRestaurant = sourceData.value.data.onCreateRestaurant;
      if(newRestaurant){
        if(this.restaurants.some(r => r.id == newRestaurant.id)) return;
        this.restaurants = [newRestaurant, ...this.restaurants];
      }
    });
    
  },
  methods: {  // Fix the typo here
    async getData(){
      try {
        this.loading = true;
        const response = await API.graphql(graphqlOperation(listRestaurants));
        this.restaurants = response.data.listRestaurants.items;
      }
      catch(error){
        console.log('Error loading restaurants: ', error);
      }
      finally {
        this.loading = false;
      }
    },

    async createRestaurant(){
      const { name, description, city } = this.form;
      if(!name || !description || !city) return;

      const restaurant = { name, description, city, clientId: this.clientId };

      try {
        const response = await API.graphql(graphqlOperation(createRestaurant, { input: restaurant }));
        console.log('Item created');
        this.restaurants = [...this.restaurants, response.data.createRestaurant];
        this.form = { name: '', description: '', city: '' };
      } 
      catch(error) {
        console.error('Error creating restaurant:', error);
      }
    }
  }
};

</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
