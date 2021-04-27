<template>
<div>
  <h1>rwr</h1>
  <div v-for="restaurant of restaurants" :key="restaurant.id">
    {{restaurant.name}}
  </div>


    <form v-on:submit.prevent>
      <div>
        <label>Name: </label>
        <input v-model='form.name' class='input' />
      </div>
      ...
      <button @click='createRestaurant' class='button'>Create</button>
    </form>



  </div>
</template>
<script>
import { API, graphqlOperation } from 'aws-amplify';
import { listRestaurants } from './graphql/queries';
import { createRestaurant, deleteRestaurant } from './graphql/mutations';
import {onCreateRestaurant ,onDeleteRestaurant} from './graphql/subscriptions';

export default {
  name: 'App',
  data() {
    return {
      restaurants: [],
      form: { }

    }
  },
  created() {
    this.getData();


    API.graphql(graphqlOperation(onCreateRestaurant)).subscribe((sourceData) => {
      const newRestaurant = sourceData.value.data.onCreateRestaurant
      if (newRestaurant) {
        // skip our own mutations and duplicates
        if (this.restaurants.some(r => r.id == newRestaurant.id)) return;
        this.restaurants = [newRestaurant, ...this.restaurants];
     } 
    });

   API.graphql(graphqlOperation(onDeleteRestaurant)).subscribe((sourceData) => {
      const deletedRestaurant = sourceData.value.data.onDeleteRestaurant
      if (deletedRestaurant) {
        // skip our own mutations and duplicates
        if (this.restaurants.some(r => r.id == deletedRestaurant.id)) return;
        this.restaurants = [deletedRestaurant, ...this.restaurants];
     } 
    });



  },
  methods: {
    async getData() {
      try {
        const response = await API.graphql(graphqlOperation(listRestaurants));
        this.restaurants = response.data.listRestaurants.items;
      }
      catch (error) {
        console.log('Error loading restaurants...', error);
      }
    },

    async createRestaurant() {
      const { name, description, city } = this.form
      if (!name || !description || !city) return;
    
      const restaurant = { name, description, city };
      try {
        const response = await API.graphql(graphqlOperation(createRestaurant, { input: restaurant }))
        console.log('Item created!')
        this.restaurants = [...this.restaurants, response.data.createRestaurant];
        this.form = { name: '', description: '', city: '' };
      } catch (error) {
        console.log('Error creating restaurant...', error)
      }
    },
    async deleteRestaurant(restaurant) {
      if(restaurant) {
        try {
          const {id} =restaurant;
          await API.graphql(graphqlOperation(deleteRestaurant, { input: {id:id}}))
          console.log("item deleted")
          this.restaurants = this.restaurants.filter((r) => r.id != restaurant.id);

        }catch (error) {
          console.log("rror deleting restaurant", error);
        }
      }
    }

  }
}
</script>