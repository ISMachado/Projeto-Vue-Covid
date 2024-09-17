<template>
  <div id="app">
    <nav>
      <div class="container">
        <img src="@/assets/logo.png" alt="Logo" />
      </div>
    </nav>
    <div class="container">
      <!-- Header com título e descrição -->
      <header>
        <div class="header-content">
          <div class="row">
            <div class="col-lg-6 pt-lg-5">
              <h1>Conheça o Covidômetro</h1>
              <p>
                Fique atualizado com veracidade e transparência. O Covidômetro é uma
                ferramenta para mostrar ao usuário os dados mais recentes de casos e
                óbitos  relacionados à pandemia de COVID-19 ao redor do mundo.
              </p>
            </div>
            <div class="col-lg-6">
              <img src="@/assets/doctor-image.png" alt="Ilustração de Médicos" class="img" />
            </div>
          </div>
          <div class="text-center">
            <div>
              <h2>Filtrar dados sobre um país</h2>
            </div>
  
            <!-- Filtro de país -->
            <div class="search-container">
              <input
                type="text"
                v-model="searchTerm"
                placeholder="Digite o nome de um país"
              />
            </div>
        
            <div>
              <!-- Botão para ordenar países por quantidade de casos -->
              <button @click="sortCountries">
                {{
                  sortByCases === "asc"
                    ? "Ordenar por Mais Casos"
                    : "Ordenar por Menos Casos"
                }}
              </button>
              
              <!-- Botão para ordenar países alfabeticamente -->
              <button @click="sortAlphabetically">
                {{
                  sortAlphabetical === "asc"
                    ? "Ordenar por Nome (Z-A)"
                    : "Ordenar por Nome (A-Z)"
                }}
              </button>
            </div>
        
            <!-- Lista de países -->
            <div class="country-list">
              <CountryList
                :countries="filteredCountries"
                @selectCountry="fetchCountryDetails"
              />
            </div>
        
            <!-- Detalhes de um país -->
            <div v-if="selectedCountry" class="country-details">
              <h2>Detalhes de {{ selectedCountry.name }}</h2>
              <ul>
                <li v-for="report in countryDetails" :key="report.date">
                  {{ report.date }} - Casos: {{ report.confirmed }} - Mortes:
                  {{ report.deaths }}
                </li>
              </ul>
            </div>
          </div>
          </div>
      </header>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import CountryList from "./components/CountryList.vue";

export default {
  name: "App",
  components: {
    CountryList,
  },
  data() {
    return {
      countries: [],
      searchTerm: "",
      selectedCountry: null,
      countryDetails: [],
      sortByCases: false,
      sortAlphabetical: false,
    };
  },
  computed: {
    filteredCountries() {
      let filtered = this.countries.filter((country) =>
        country.name.toLowerCase().includes(this.searchTerm.toLowerCase())
      );

      if (this.sortAlphabetical) {
        // Ordena alfabeticamente A-Z ou Z-A
        filtered.sort(
          (a, b) =>
            this.sortAlphabetical === "asc"
              ? a.name.localeCompare(b.name) // A-Z
              : b.name.localeCompare(a.name) // Z-A
        );
      } else if (this.sortByCases) {
        // Ordena por número de casos
        filtered.sort(
          (a, b) =>
            this.sortByCases === "asc"
              ? a.cases - b.cases // Menos casos
              : b.cases - a.cases // Mais casos
        );
      }

      return filtered;
    },
  },
  methods: {
    fetchCountries() {
      axios
        .get("https://covid-api.com/api/reports")
        .then((response) => {
          const data = response.data.data;
          const uniqueCountries = [];

          // Filtra os dados, mantendo apenas países com código ISO único
          data.forEach((report) => {
            if (
              !uniqueCountries.find(
                (country) => country.iso === report.region.iso
              )
            ) {
              uniqueCountries.push({
                name: report.region.name,
                cases: report.confirmed,
                deaths: report.deaths,
                iso: report.region.iso,
              });
            }
          });

          this.countries = uniqueCountries;
        })
        .catch((error) => console.error(error));
    },
    fetchCountryDetails(iso) {
      axios
        .get(`https://covid-api.com/api/reports?iso=${iso}`)
        .then((response) => {
          this.countryDetails = response.data.data.slice(0, 5);
          this.selectedCountry = this.countries.find(
            (country) => country.iso === iso
          );
        })
        .catch((error) => console.error(error));
    },
    sortCountries() {
      // Alterna entre ordenação crescente e decrescente
      this.sortByCases = this.sortByCases === "asc" ? "desc" : "asc";
      this.sortAlphabetical = false; // Desativa a ordenação alfabética
    },
    sortAlphabetically() {
      // Alterna entre ordenação crescente e decrescente
      this.sortAlphabetical = this.sortAlphabetical === "asc" ? "desc" : "asc";
      this.sortByCases = false; // Desativa a ordenação por casos
    },
  },
  created() {
    this.fetchCountries();
  },
};
</script>

<style>
/* Reset */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: "Spectral", serif;
  background-color: #f7f6f6;
}

nav {
  background-color: #fff;
  padding: 20px;
  margin-bottom: 30px;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
  align-items: center;
}

.header-content h1 {
  font-family: "Spectral";
  font-size: 48px;
  font-weight: 700;
  color: #333;
}

.header-content p {
  font-size: 1.2rem;
  margin-top: 10px;
  color: #666;
}

.header-content .img {
  padding-left: 160px;
}

.header-content h2 {
  font-family: "Spectral";
  font-size: 30px;
  font-weight: 700;
  color: #333;
}

/* Caixa de busca */
.search-container {
  margin-bottom: 20px;
}

.search-container input {
  padding: 10px 20px;
  width: 100%;
  max-width: 600px;
  border: 1px solid #ddd;
  border-radius: 30px;
  font-size: 1rem;
  box-shadow: 0px 2px 4px rgba(0, 0, 0, 0.1);
}

/* Lista de países */
.country-list {
  background-color: #fff;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
}

.country-list ul {
  list-style-type: none;
  padding: 0;
}

.country-list li {
  padding: 20px;
  margin-bottom: 15px;
  background-color: #fafafa;
  border-radius: 10px;
  box-shadow: 0px 2px 4px rgba(0, 0, 0, 0.1);
  display: flex;
  justify-content: space-between;
  align-items: center;
  transition: background-color 0.3s ease;
}

.country-list li:hover {
  background-color: #f1f1f1;
}

.country-list .country-info {
  display: flex;
  justify-content: space-between;
  width: 100%;
}

.country-info div {
  text-align: left;
}

/* País - dados */
.country-list .country-name {
  font-weight: bold;
  font-size: 1.2rem;
  color: #333;
}

.country-list .country-stats {
  font-size: 0.9rem;
  color: #666;
}

.country-list .fatality-rate {
  font-weight: bold;
  color: #ff4c4c;
}

/* Detalhes do país selecionado */
.country-details {
  margin-top: 30px;
  text-align: left;
}

.country-details h2 {
  margin-bottom: 10px;
}

.country-details ul {
  list-style-type: none;
}

.country-details li {
  padding: 10px;
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0px 2px 4px rgba(0, 0, 0, 0.1);
  margin-bottom: 10px;
}

button {
  padding: 10px 20px;
  margin-left: 10px;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 1rem;
  transition: background-color 0.3s ease;
}

button:hover {
  background-color: #45a049;
}
</style>
