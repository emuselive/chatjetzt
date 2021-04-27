<template>
  <div>
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
import { createRestaurant } from './graphql/mutations';

export default {
  name: 'app',
  data() {
    return {
      form: { },
    }
  },
  methods: {
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
  }
}
</script>