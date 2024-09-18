<template>
  <div>
    <ul>
      <li
        v-for="country in countries"
        :key="country.iso + country.name"
        @click="selectCountry(country.iso)"
      >
        Total de casos: {{ country.cases }} - {{ country.name }} - Mortes:
        {{ country.deaths }}
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  props: ["countries"],
  methods: {
    selectCountry(iso) {
      this.$emit("selectCountry", iso);
    },
  },
};
</script>

<style>
ul {
  list-style-type: none;
  padding: 0;
}

li {
  padding: 10px;
  background: #f4f4f4;
  margin-bottom: 5px;
  cursor: pointer;
}

li:hover {
  background: #e4e4e4;
}
</style>
