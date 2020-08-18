<template>
  <div id="app">
    <label for="raceSelect">Select Race:</label>
    <select name="raceSelect" id="raceSelect" v-model="race">
      <option value="argonian">Argonian</option>
      <option value="breton">Breton</option>
      <option value="dunmer">Dark Elf</option>
      <option value="altmer">High Elf</option>
      <option value="imperial">Imperial</option>
      <option value="khajiit">Khajiit</option>
      <option value="nord">Nord</option>
      <option value="orc">Orc</option>
      <option value="redguard">Redguard</option>
      <option value="bosmer">Wood Elf</option>
    </select>
    <select name="gender" id="genderSelect" v-model="gender">
      <option value="male">Male</option>
      <option value="female">Female</option>
    </select>
    <span v-if="hasSurname">
      <label for="toggleSurname">Generate Family Name:</label>
      <input type="checkbox" id="toggleSurname" v-model="toggleSurname">
    </span>
    <button v-on:click="generateName()">Generate</button>
    <h2 id="name">{{ fullName }}</h2>
  </div>
</template>

<script>

export default {
  name: 'App',
  components: {
  },
  data: () => {
    return {
      fullName: '',
      race: 'argonian',
      gender: 'male',
      isReady: false,
      toggleSurname: true,
      hasSurname: true
    }
  },
  methods: {
    generateName() {
      const names = require(`./names/${this.race}.json`);
      // Hide toggle family name button if the selected race does not have family names
      this.toggleSurname = names.hasSurname;
      const name = this.getRandomName(this.gender == 'male' ? names.male : names.female);
      let surname = '';
      if(this.toggleSurname) {
        switch(this.race) {
          // Orc family name is based on the parent's name & gender based
          case 'orc':
            surname = this.getOrcSurname(names);
            break;
          // Redguard family names can be based on relatives or birthplace
          case 'redguard':
            surname = this.getRedguardSurname(names);
            break;
          default:
            surname = this.getRandomName(names.surnames)
            break;
        }
      }
      this.fullName = `${name} ${surname}`;
    },

    getRandomName(array) {
      const random = Math.floor(Math.random() * array.length);
      return array[random];
    },

    getOrcSurname(names) {
      const prefix = (this.gender == 'male' ? "gro-" : "gra-");
      return `${prefix}${this.getRandomName(this.gender == 'male' ? names.male : names.female)}`;
    },

    getRedguardSurname(names) {
      const prefixes = ["at", "af", "al"];
      const random = Math.floor(Math.random() * prefixes.length);
      // 2 equals name based on birthplace
      const suffix = this.getRandomName(random === 2 ? names.birthplaces : names.male.concat(names.female));
      return `${prefixes[random]}-${suffix}`;
    }
  },
  computed: {

  },
  watch: {

  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
