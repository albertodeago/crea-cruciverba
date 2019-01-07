<template>
  <v-container>
    <v-layout
      text-xs-center
      wrap
    >

      <v-flex xs12>
        <v-checkbox label="Qualsiasi lunghezza" v-model="undefinedLength"></v-checkbox>
        <v-text-field label="Inserisci una parola con degli * dove non sai che lettere mettere" v-model="pattern"
        ></v-text-field>
        <v-btn @click="searchWord">Search</v-btn>
      </v-flex>


      <!-- <v-flex mb-5 xs12>
        <h2 class="headline font-weight-bold mb-3">What's next?</h2>
        <v-layout justify-center>
          <a
            v-for="(next, i) in whatsNext"
            :key="i"
            :href="next.href"
            class="subheading mx-3"
            target="_blank"
          >
            {{ next.text }}
          </a>
        </v-layout>
      </v-flex> -->

    </v-layout>
  </v-container>
</template>

<script>
  import dictIta from "../assets/ita-dict.json";

  export default {
    data: () => ({
      dictionary: null,
      pattern: "",
      undefinedLength: false,

      whatsNext: [
        {
          text: 'Explore components',
          href: 'https://vuetifyjs.com/components/api-explorer'
        },
        {
          text: 'Select a layout',
          href: 'https://vuetifyjs.com/layout/pre-defined'
        },
        {
          text: 'Frequently Asked Questions',
          href: 'https://vuetifyjs.com/getting-started/frequently-asked-questions'
        }

      ]
    }),

    methods: {
      searchWord() {
        console.log(this.pattern);
        const availableWords = [];
        const map = {};
        const letters = this.pattern.split("");
        const length = letters.length;
        let counter = 0;

        // filter by length to relieve a bit
        const words = this.undefinedLength ? this.dictionary : this.dictionary.filter( a => a.length === length);
        console.log("remained " + words.length + " words");

        letters.forEach((letter, index) => {
          if(letter !== "_" && letter !== "*") {
            map[index] = letter;
          }
        });

        words.forEach( word => {
          let mismatch = false;
          Object.keys(map).forEach( key => {
            if(word[key] !== letters[key]) {
              mismatch = true;
            }
          });

          if(!mismatch)
            availableWords.push(word);

          ++counter;
        });

        console.log("checked for " + counter + " words");
        console.log(availableWords);
      }
    },

    created() {
      this.dictionary = dictIta.dictionary;
    }
  }
</script>

<style>

</style>
