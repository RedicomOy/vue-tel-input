<template>
  <div id="app">
    <b-card title="Telephone Input Vue"
            sub-title="made with &amp;#x2764; by Steven.">
      <b-row style="margin-top: 20px;">
        <b-col md="4"
               offset="4">
          <vue-tel-input @onInput="onInput">
          </vue-tel-input>
        </b-col>
      </b-row>
      <b-row>
        <b-col md="4" offset="4">
          <b-btn v-b-modal.modal1>Launch demo modal</b-btn>
          <b-modal id="modal1" title="Bootstrap-Vue" @focusOut="console.log('out')">
            <vue-tel-input @onInput="onInput">
            </vue-tel-input>

            <b-dropdown variant="outline-secondary">
              <template slot="button-content">
                button
              </template>
              <b-dropdown-header @click.prevent>
                <b-form-input v-model="search" placeholder="Search by name, ISO2 or country code" />
              </b-dropdown-header>
              <b-dropdown-item key="1">
                <strong>name</strong>
              </b-dropdown-item>
            </b-dropdown>

          </b-modal>
        </b-col>
      </b-row>
      <b-row v-if="phone.number"
             style="margin-top: 20px;">
        <b-col md="4"
               offset="4">
          <code>
            <span>Number:
              <strong>{{ phone.number }}</strong>,&nbsp;
            </span>
            <span>Is valid:
              <strong>{{ phone.isValid }}</strong>,&nbsp;
            </span>
            <span>Country:
              <strong>{{ phone.country }}</strong>
            </span>
          </code>
        </b-col>
      </b-row>
    </b-card>
  </div>

</template>

<script>
export default {
  name: "app",
  data() {
    return {
      phone: {
        number: "",
        isValid: false,
        country: undefined
      }
    };
  },
  methods: {
    onInput({ number, isValid, country }) {
      this.phone.number = number;
      this.phone.isValid = isValid;
      this.phone.country = country;
    }
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
