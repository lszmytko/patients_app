<template>
  <div class="Card">
    <h3>{{ name }} {{ lastName }}</h3>
    <h4>Age: {{ age }}</h4>
    <h5>{{medicines.length ? 'Medicine list' : 'No medicines'}}</h5>
    <ul v-if="medicines.length">
      <li v-for="medicine in medicines" :key="medicine.id">
        {{ medicine.medicationName }}
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: "Card",
  props: ["name", "lastName", "medicines", "age"],
};
</script>

<style scoped>
.Card {
  width: 300px;
  background: rgb(142, 202, 230);
  border-radius: 1rem;
  padding: 1rem;
}

h3 {
  letter-spacing: 0.1.5rem;
  font-weight: 700;
  margin-bottom: .75rem;
  text-align: center;
}

h4 {
  margin-bottom: 1rem;
}

h5{
  margin-bottom: .5rem;
}


ul{
  list-style: none;
}

li {
  font-size: 0.75rem;
  margin-bottom: 0.25rem;
}
</style>