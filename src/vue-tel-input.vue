<template>
  <b-row class="vue-tel-input justify-content-md-center">
    <b-col col="col"
           md="12">
      <b-input-group>
        <b-input-group-prepend>
          <b-dropdown variant="outline-secondary" @hide="hideDropdown" @shown="shownDropdown">
            <template slot="button-content">
              <img :src="activeCountry.icon"
                  style="width: 25px; margin-right: 5px" />
            </template>
            <b-dropdown-header @click="clickSearch">
              <b-form-input v-model="search" id="search" placeholder="Search by name, ISO2 or country code" @blur.native="blurSearch" />
            </b-dropdown-header>
            <b-dropdown-item v-for="pb in preferredCountries"
                            :key="`pref-${pb['iso2']}`"
                            @click="choose(pb)">
              <img :src="pb.icon"
                  style="width: 25px; margin-right: 5px" />
              <strong>{{ pb.name }} </strong>
              <span>+{{ pb.dialCode }}</span>
            </b-dropdown-item>
            <b-dropdown-divider></b-dropdown-divider>
            <b-dropdown-item v-for="pb in filteredCountries"
                            :key="pb['iso2']"
                            @click="choose(pb)">
              <img :src="pb.icon"
                  style="width: 25px; margin-right: 5px" />
              <strong>{{ pb.name }} </strong>
              <span>+{{ pb.dialCode }}</span>
            </b-dropdown-item>
          </b-dropdown>
        </b-input-group-prepend>
        <b-form-input
          v-model="phone"
          placeholder="Enter your phone number"
          ref="phone-input"
          :state="state"
          :formatter="format"
          @input="onInput"
          @keydown.native.enter.prevent="onEnter"
        >
        </b-form-input>
      </b-input-group>
    </b-col>
  </b-row>
</template>

<script>
import { format, asYouType, isValidNumber } from "libphonenumber-js";
import allCountries from "./assets/all-countries";
import getCountry from "./assets/default-country";

//hide
//bvEvt BvEvent object. Call bvEvt.preventDefault()

export default {
  name: "vue-tel-input",
  props: {
    value: {
      type: String
    },
    name: {
      type: String
    }
  },
  mounted() {
    getCountry().then(res => {
      this.activeCountry =
        allCountries.find(country => country.iso2 === res) || allCountries[0];
    });
  },
  created() {
    if (this.value) {
      this.phone = this.value;
    }
  },
  data() {
    return {
      phone: "",
      allCountries,
      activeCountry: { iso2: "" },
      preferredCountryCodes: [
        "FI",
        "SE",
        "NO",
        "DA",
        "DE",
        "FR",
        "GB",
        "US",
        "ES",
        "IT",
        "RU",
        "CN",
        "JP"
      ],
      search: "",
      searchActive: false
    };
  },
  computed: {
    mode() {
      if (!this.phone) {
        return "";
      }
      if (this.phone[0] === "+") {
        return "code";
      }
      if (this.phone[0] === "0") {
        return "prefix";
      }
      return "normal";
    },
    formattedResult() {
      // Calculate phone number based on mode
      if (!this.mode || !this.allCountries) {
        return "";
      }
      let phone = this.phone;
      if (this.mode === "code") {
        // If user manually type the country code
        const formatter = new asYouType(); // eslint-disable-line
        formatter.input(this.phone);

        // Find inputted country in the countries list
        this.activeCountry =
          this.allCountries.find(
            ele => ele.iso2.toUpperCase() === formatter.country
          ) || this.activeCountry;
      } else if (this.mode === "prefix") {
        // Remove the first '0' if this is a '0' prefix number
        // Ex: 0432421999
        phone = this.phone.slice(1);
      }
      return format(
        phone,
        this.activeCountry && this.activeCountry.iso2,
        "International"
      );
    },
    state() {
      return isValidNumber(this.formattedResult);
    },
    response() {
      const number =
        this.formattedResult && this.formattedResult.replace(/ /g, "");
      return {
        number,
        isValid: this.state,
        country: this.activeCountry.name
      };
    },
    preferredCountries() {
      if (this.search) {
        return this.allCountries
          .filter(c => this.preferredCountryCodes.indexOf(c.iso2) > -1)
          .filter(
            c =>
              c.name.toUpperCase().indexOf(this.search.toUpperCase()) > -1 ||
              c.iso2.toUpperCase().indexOf(this.search.toUpperCase()) > -1 ||
              c.dialCode.indexOf(this.search) > -1
          );
      }

      return this.allCountries.filter(
        c => this.preferredCountryCodes.indexOf(c.iso2) > -1
      );
    },
    filteredCountries() {
      if (this.search) {
        return this.allCountries
          .filter(c => this.preferredCountryCodes.indexOf(c.iso2) === -1)
          .filter(
            c =>
              c.name.toUpperCase().indexOf(this.search.toUpperCase()) > -1 ||
              c.iso2.toUpperCase().indexOf(this.search.toUpperCase()) > -1 ||
              c.dialCode.indexOf(this.search) > -1
          );
      }

      return this.allCountries;
    }
  },
  watch: {
    state(value) {
      if (value) {
        // If mode is 'prefix', keep the number as user typed,
        // Otherwise format it
        if (this.mode !== "prefix") {
          this.phone = this.formattedResult;
        }
      }
    }
  },
  methods: {
    choose(country) {
      this.activeCountry = country;
      // don't send event for country selection
      // this.$emit("oninput", this.response);
    },
    format(value) {
      return new asYouType(this.activeCountry.iso2).input(value); // eslint-disable-line
    },
    onInput() {
      // Emit input event in case v-model is used in the parent
      this.$emit("input", this.response.number);

      // Emit the response, includes phone, validity and country
      this.$emit("oninput", this.response, this.name);
    },
    shownDropdown() {
      setTimeout(() => {
        this.$el.querySelector("input").focus();
      }, 300);
    },
    hideDropdown(bvEvt) {
      if (this.searchActive) {
        bvEvt.preventDefault();
      }
    },
    clickSearch() {
      this.searchActive = true;
    },
    blurSearch() {
      this.searchActive = false;
    },
    onEnter() {
      this.$emit("enter", this.response.number);
    }
  }
};
</script>

<style src="bootstrap/dist/css/bootstrap.css"></style>
<style>
.dropdown-menu.show {
  max-height: 300px;
  overflow: scroll;
}
</style>