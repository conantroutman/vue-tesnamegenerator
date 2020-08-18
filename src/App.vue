<template>
  <div id="app">
    <h1>TES Random Name Generator</h1>
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
    <select name="number" id="number" v-model="numberToGenerate">
      <option value="1">1</option>
      <option value="5">5</option>
      <option value="10">10</option>
      <option value="20">20</option>
    </select>
    <button v-on:click="generateName()">Generate</button>
    <ul class="results" >
      <li v-for="item in generatedNames" v-bind:key="item.id">
        {{ item | capitalize }}
      </li>
    </ul>
  </div>
</template>

<script>

export default {
  name: 'App',
  components: {
  },
  data: () => {
    return {
      generatedNames: [],
      race: 'argonian',
      gender: 'male',
      isReady: false,
      toggleSurname: true,
      hasSurname: true,
      numberToGenerate: 1,
    }
  },
  methods: {
    generateName() {
      // Reset the array of names so that a new list is generated each time
      this.generatedNames = [];

      // Load the JSON file containing the names
      const list = require(`./names/${this.race}.json`);

      // Hide toggle family name button if the selected race does not have family names
      //this.hasSurname = names.hasSurname;

      for(let i = 0; i < this.numberToGenerate; i++) {
        let name = '';
        if (this.race === 'khajiit') {
          name = this.getKhajiitName(list);
        } else {
          name = this.getRandomName(this.gender == 'male' ? list.male : list.female)
        }

        let surname = '';
        if(this.toggleSurname) {
          switch(this.race) {
            // Orc family name is based on the parent's name & gender based
            case 'orc':
              surname = this.getOrcSurname(list);
              break;
            // Redguard family names can be based on relatives or birthplace
            case 'redguard':
              surname = this.getRedguardSurname(list);
              break;
            default:
              surname = this.getRandomName(list.surnames)
              break;
          }
        }
        //this.fullName = `${name} ${surname}`;
        this.generatedNames.push(`${name} ${surname}`);
      }
    },

    // Grabs a random array element and returns it
    getRandomName(array) {
      const random = Math.floor(Math.random() * array.length);
      return array[random];
    },

    // Returns an Orcish family name
    getOrcSurname(names) {
      const prefix = (this.gender == 'male' ? "gro-" : "gra-");
      return `${prefix}${this.getRandomName(this.gender == 'male' ? names.male : names.female)}`;
    },

    // Returns a Redguard family nmame
    getRedguardSurname(names) {
      const prefixes = ["at", "af", "al"];
      const random = Math.floor(Math.random() * prefixes.length);
      // 2 equals name based on birthplace
      const suffix = this.getRandomName(random === 2 ? names.birthplaces : names.male.concat(names.female));
      return `${prefixes[random]}-${suffix}`;
    },

    // Returns a Khajiit name with added prefix/suffix. Might want to make the prefix/suffix generation input based rather than rng based in the future.
    getKhajiitName(list) {

      // Males are more likely to have an honorific title according to UESP
      const chanceTitle = (this.gender === 'male' ? 0.5 : 0.15);
      // Prefixes are more common than suffixes
      const prefixBias = 0.75;

      const prefixesM = ["Dar", "Do", "Dro", "J", "Jo", "M", "R", "Ri", "S"]
      const prefixesF = ["Daro", "Do", "Dra", "L", "Ko", "M", "R", "Ri", "S"]
      const suffixesM = ["dar", "do", "dro", "ja", "jo", "ma", "ra", "ri", "sa"]
      const suffixesF = ["daro", "do", "dra", "la", "ko", "ma", "ra", "ri", "sa"]

      let name = this.getRandomName(this.gender === 'male' ? list.male : list.female);

      if ((Math.random() * 1) <= chanceTitle) {
        const isPrefix = ((Math.random() * 1) <= prefixBias ? true : false);

        if (isPrefix) {
          const prefix = (this.gender === 'male' ? this.getRandomName(prefixesM) : this.getRandomName(prefixesF));
          name = `${prefix}'${name.toLowerCase()}`;
        } else {
          const suffix = (this.gender === 'male' ? this.getRandomName(suffixesM) : this.getRandomName(suffixesF));
          name = `${name}-${suffix}`;
        }
      }

      return name;
    }
  },
  filters: {
    // Auto capitalize the first letter. Shamelessly stolen from Vue.js documentation.
    capitalize: function (value) {
      if (!value) return ''
      value = value.toString()
      return value.charAt(0).toUpperCase() + value.slice(1)
  }
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
.results {
  list-style: none;
  font-size: 1.75rem;
}

.results li {
  margin-bottom: .25rem;
}
</style>
