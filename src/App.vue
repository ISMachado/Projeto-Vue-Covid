<template>
  <div id="app">
    <nav>
      <div class="container">
        <img src="@/assets/logo.png" alt="Logo" />
      </div>
    </nav>
    <div class="container">
      <header>
        <div class="header-content">
          <div class="row">
            <div class="col-lg-6 pt-lg-5">
              <h1>Conheça o Covidômetro</h1>
              <p>
                Fique atualizado com veracidade e transparência. O Covidômetro é
                uma ferramenta para mostrar ao usuário os dados mais recentes de
                casos e óbitos relacionados à pandemia de COVID-19 ao redor do
                mundo.
              </p>
            </div>
            <div class="col-lg-6">
              <img
                src="@/assets/doctor-image.png"
                alt="Ilustração de Médicos"
                class="img"
              />
            </div>
          </div>
          <div class="text-center country-spacing">
            <div class="search-container">
              <div>
                <h2>Filtrar dados sobre um país</h2>
              </div>
              <div class="search-input-wrapper px-5">
                <i class="fa-solid fa-magnifying-glass"></i>
                <input
                  type="text"
                  v-model="searchTerm"
                  placeholder="Digite o nome do país"
                />
              </div>
              <div class="pt-4 button-group">
                <button @click="sortCountries">
                  {{
                    sortByCases === "asc"
                      ? "Ordenar por Mais Casos"
                      : "Ordenar por Menos Casos"
                  }}
                </button>

                <button @click="sortAlphabetically">
                  {{
                    sortAlphabetical === "asc"
                      ? "Ordenar por Nome (Z-A)"
                      : "Ordenar por Nome (A-Z)"
                  }}
                </button>
              </div>
            </div>

            <div class="country-list pb-5">
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
  font-family: "Spectral", serif !important;
  background-color: #fff9f9 !important;
}

nav {
  background-color: #fff;
  padding: 20px;
  margin-bottom: 30px;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
  align-items: center;
}

.header-content h1 {
  font-size: 48px;
  font-weight: 700;
  color: #4a4543;
}

.header-content p {
  font-size: 18px;
  margin-top: 10px;
  color: #575555;
}

.header-content .img {
  padding-left: 160px;
}

.header-content h2 {
  font-size: 30px;
  font-weight: 700;
  color: #4a4543;
}

.country-spacing {
  padding-right: 180px;
  padding-left: 180px;
}

/* Caixa de busca */
.search-container {
  margin-bottom: 20px;
  background-color: #fff;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
}

.search-input-wrapper {
  position: relative;
}

.search-input-wrapper input {
  padding: 10px 50px 10px 40px;
  width: 100%;
  border: none;
  border-bottom: 2px solid #f0615a;
  border-radius: 0;
  font-size: 16px;
  transition: border-color 0.3s ease;
  align-items: center;
}

.search-input-wrapper input:focus {
  outline: none;
}

.search-input-wrapper i {
  position: absolute;
  left: 60px;
  top: 52%;
  transform: translateY(-50%);
  font-size: 18px;
  color: #f0615a;
}

.search-input-wrapper input::placeholder {
  color: #aaa;
}

.button-group button {
  margin-right: 15px;
  margin-bottom: 10px;
  border-radius: 8px;
}

button {
  padding: 10px 20px;
  margin-left: 10px;
  background-color: #f0615a;
  color: white;
  border: none;
  cursor: pointer;
  font-size: 1rem;
}

/* Lista de países */
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
</style>
