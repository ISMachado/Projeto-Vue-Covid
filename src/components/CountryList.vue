<template>
  <div class="country-list">
    <div v-for="country in countries" :key="country.iso + country.name" class="country-item">
      <div class="country-name pb-4">{{ country.name }}</div>
      <div class="country-info">
        <div class="country-stats">
          <div class="country-cases">
            <span class="label">Total de casos</span>
            <span class="value">{{ country.cases.toLocaleString() }}</span>
          </div>
          <div class="country-deaths">
            <span class="label">Mortes</span>
            <span class="value">{{ country.deaths.toLocaleString() }}</span>
          </div>
          <div class="country-fatality-rate">
            <span class="label">Fatalidade</span>
            <span class="value">{{ ((country.deaths / country.cases) * 100).toFixed(2) }}%</span>
          </div>
        </div>
      </div>
    </div>
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

<style scoped>
.country-list {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.country-item {
  background-color: #fff;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
  transition: background-color 0.3s ease;
}

.country-item:hover {
  background-color: #f9f9f9;
}

.country-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.country-name {
  font-size: 30px;
  font-weight: bold;
  color: #4a4543;
  flex: 1;
}

.country-stats {
  display: flex;
  flex: 2;
  justify-content: space-around;
}

.country-stats .label {
  display: block;
  font-size: 18px;
  color: #575555;
  font-weight: 500;
}

.country-stats .value {
  font-size: 25px;
  font-weight: bold;
  color: #f0615a;
}

.country-cases,
.country-deaths,
.country-fatality-rate {
  text-align: center;
}
</style>
