<template>
  <div id="app">
    <h1>Country Management</h1>
    {{countries}}
    <select v-model="selectedCountryId" @change="fetchCountryDetails">
      <option v-if="countries.length === 0">No countries available</option>
      <option
        v-for="(country, index) in countries"
        :key="index"
        :value="index + 1"
      >
        {{ country.name }}
      </option>
    </select>

    <div v-if="countryDetails">
      <h2>Country Details</h2>
      <p><strong>Name:</strong> {{ countryDetails.name }}</p>
      <p><strong>Rank:</strong> {{ countryDetails.rank }}</p>
      <p><strong>Continent:</strong> {{ countryDetails.continent }}</p>
      <img
        :src="`http://localhost:8080/${countryDetails.flag}`"
        v-if="countryDetails.flag"
        alt="Country Flag"
        width="200"
      />
    </div>

    <h2>Add a New Country</h2>
    <form @submit.prevent="addCountry" enctype="multipart/form-data">
      <label>
        Country Name:
        <input
          v-model="newCountry.name"
          type="text"
          :class="{ error: errors.name }"
        />
        <p v-if="errors.name">{{ errors.name }}</p>
      </label>
      <label>
        Continent:
        <select v-model="newCountry.continent">
          <option
            v-for="continent in uniqueContinents"
            :key="continent"
            :value="continent"
          >
            {{ continent }}
          </option>
        </select>
      </label>
      <label>
        Rank:
        <input
          v-model="newCountry.rank"
          type="number"
          :class="{ error: errors.rank }"
        />
        <p v-if="errors.rank">{{ errors.rank }}</p>
      </label>
      <label>
        Country Flag:
        <input
          type="file"
          @change="handleFileUpload"
          :class="{ error: errors.flag }"
        />
        <p v-if="errors.flag">{{ errors.flag }}</p>
      </label>
      <button type="submit">Add Country</button>
    </form>

    <div v-if="submitMessage">{{ submitMessage }}</div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      countries: [],
      selectedCountryId: null,
      countryDetails: null,
      newCountry: {
        name: "",
        continent: "",
        rank: "",
        flag: null,
      },
      uniqueContinents: [],
      errors: {},
      submitMessage: "",
    };
  },
  mounted() {
    this.loadCountries();
  },
  methods: {
    async loadCountries() {
      try {
        const response = await axios.get("http://localhost:8080/countries");
        this.countries = response.data;
        this.uniqueContinents = [
          ...new Set(this.countries.map((c) => c.continent)),
        ];
      } catch (error) {
        console.error("Error loading countries:", error);
      }
    },
    async fetchCountryDetails() {
      try {
        const response = await axios.get(
          `http://localhost:8080/country/${this.selectedCountryId}`
        );
        this.countryDetails = response.data;
      } catch (error) {
        console.error("Error fetching country details:", error);
      }
    },
    handleFileUpload(event) {
      const file = event.target.files[0];
      if (
        file &&
        (file.type === "image/jpeg" || file.type === "image/png") &&
        file.size <= 4 * 1024 * 1024
      ) {
        this.newCountry.flag = file;
        this.errors.flag = null;
      } else {
        this.errors.flag = "Only JPG/PNG files up to 4MB are allowed";
      }
    },
    async addCountry() {
      this.errors = {};

      if (this.newCountry.name.length < 3 || this.newCountry.name.length > 20) {
        this.errors.name = "Country name must be between 3 and 20 characters";
        return;
      }
      if (!this.newCountry.rank || isNaN(this.newCountry.rank)) {
        this.errors.rank = "Rank must be a valid number";
        return;
      }

      try {
        const formData = new FormData();
        formData.append("name", this.newCountry.name);
        formData.append("continent", this.newCountry.continent);
        formData.append("rank", this.newCountry.rank);
        if (this.newCountry.flag) formData.append("flag", this.newCountry.flag);

        const response = await axios.post(
          "http://localhost:8080/country",
          formData,
          {
            headers: {
              "Content-Type": "multipart/form-data",
            },
          }
        );

        this.submitMessage = response.data.message;
        this.newCountry = { name: "", continent: "", rank: "", flag: null };
        this.loadCountries(); // Refresh the list without page reload
      } catch (error) {
        if (error.response && error.response.status === 400) {
          this.errors = error.response.data.errors || {};
        }
      }
    },
  },
};
</script>

<style>
.error {
  border: 2px solid red;
}
</style>
