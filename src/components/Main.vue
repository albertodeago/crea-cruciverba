<template>
  <v-container>
    <v-layout
      text-xs-center
      wrap
    >

      <!-- Search -->
      <v-flex xs8>
        <v-layout>
          <v-checkbox label="Qualsiasi lunghezza" v-model="undefinedLength"></v-checkbox>
          <v-text-field label="Inserisci una parola con degli * dove non sai che lettere mettere" v-model="pattern"></v-text-field>
          <v-btn @click="searchWord">
            <v-icon>search</v-icon>
          </v-btn>
          <v-btn @click="clearWord">
            <v-icon>clear</v-icon>
          </v-btn>
        </v-layout>
      </v-flex>

      <!-- Results -->
      <v-flex offset-xs1 xs3>
        <v-layout column justify-left>
          <div class="headline">
            <span>Risultati ricerca </span>
            <span v-if="searchResults !== null">({{ searchResults.length }})</span>
          </div>
          <ul>
            <li v-for="result in searchResults" :key="result">
              <span class="risultato">{{result}}</span>
              <a target="_blank" :href="'https://www.google.com/search?q=significato+'+result">
                <v-icon>search</v-icon>
              </a>
            </li>
          </ul>
        </v-layout>
      </v-flex>

      <!-- Cruciverba -->
      <v-flex xs8>
        <v-layout>
          <v-text-field type="number" label="Inserisci la lunghezza del cruciverba" v-model="cruciWidth"></v-text-field>
          <v-text-field type="number" label="Inserisci l'altezza del cruciverba" v-model="cruciHeight"></v-text-field>
          <v-btn @click="updateCruci">Aggiorna</v-btn>
        </v-layout>

        <v-layout wrap class="cruciverba">
          <div class="row" v-for="(row, i) in scheme" :key="'row-'+i">
            <v-layout>
              <div class="cell" v-for="(col, j) in row" :key="'col-'+j" @click="onCellClick(i,j)" 
                    :class="[{
                      'cell-black': scheme[i][j] === null, 
                      'cell-definition-horizontal': isHorizontalDef(i,j), 
                      'cell-definition-vertical': isVerticalDef(i,j)
                    }, 'cell-'+i+'-'+j]"
              >
                <!-- {{ scheme[i][j] }} -->
                <span :id="'cell-definition-'+i+'-'+j" class="cell-definition"></span>
                <span :id="'cell-text-'+i+'-'+j" class="cell-text"></span>
              </div>
            </v-layout>
          </div>
        </v-layout>
      </v-flex>

      <!-- definizioni -->
      <v-flex xs8>
        <div>Horizontals</div>
        <v-text-field v-for="(horizontalDef, i) in horizontals" :key="'horizontal-'+i"
            :label="'Inserisci l\'orizzontale ' + (i+1)" @input="onDefinitionChange($event, horizontalDef, false)"
        ></v-text-field>

        <div>Verticals</div>
        <v-text-field v-for="(verticalDef, i) in verticals" :key="'vertical-'+i"
            :label="'Inserisci la verticale ' + (i+1)" @input="onDefinitionChange($event, verticalDef, true)"
        ></v-text-field>
      </v-flex>

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
      searchResults: null,
      cruciWidth: 8,
      cruciHeight: 6,
      scheme: [],
      horizontals: [],
      verticals: []
    }),

    computed: {
    },

    methods: {
      onDefinitionChange(value, position, isVertical) {
        console.log("changed", position);
        const splitPos = position.split('-');
        const x = parseInt(splitPos[0]);
        const y = parseInt(splitPos[1]);

        const $cruciverba = document.querySelector('.cruciverba');
        for(let i = 0; i<value.length; ++i) {
          const idQuery = isVertical ? `#cell-text-${(x + i)}-${y}` : `#cell-text-${(x)}-${y + i}`;
          const $cell = $cruciverba.querySelector(idQuery);

          if((isVertical && (x+i < this.cruciHeight && y < this.cruciWidth)) || 
             (!isVertical && (y+i < this.cruciWidth && x < this.cruciHeight))) {
            $cell.innerText = value[i];
          } else {
            console.warn("trying to write out of bound!");
          }
        }
      },

      updateDefinitions() {
        const $cruciverba = document.querySelector(".cruciverba");
        this.horizontals = [];
        this.verticals = [];

        for(let i=0; i<this.cruciHeight; ++i) {
          for(let j=0; j<this.cruciWidth; ++j) {
            const isHor = this.isHorizontalDef(i,j);
            const isVer = this.isVerticalDef(i,j);

            if(isHor) {
              this.horizontals.push(i+"-"+j);
            }
            if(isVer) {
              this.verticals.push(i+"-"+j);
            }
            if(!isHor && !isVer) {
              $cruciverba.querySelector(`#cell-definition-${i}-${j}`).innerText = "";
            }
          }
        }

        console.log(this.horizontals, this.verticals);
        this.horizontals.forEach((val, i) => {
          const cell = $cruciverba.querySelector(`#cell-definition-${val}`);
          cell.innerText = i+1;
        });
        this.verticals.forEach((val, i) => {
          const cell = $cruciverba.querySelector(`#cell-definition-${val}`);
          cell.innerText = i+1;
        });

      },

      isHorizontalDef(i, j) {
        return this.scheme && this.scheme[i] && this.scheme[i][j] && !this.scheme[i][j-1] && this.scheme[i][j+1];
      },
      isVerticalDef(i, j) {
        
        return (i === 0 && (this.scheme[i][j] && this.scheme[i+1] && this.scheme[i+1][j])) || 
              (this.scheme && this.scheme[i] && this.scheme[i][j] && this.scheme[i-1] && !this.scheme[i-1][j]) && 
              (this.scheme[i+1] && this.scheme[i+1][j]);
      },

      updateCruci() {
        
        for(let i=0; i<this.cruciHeight; ++i) {
          let row = [];

          for(let j=0; j<this.cruciWidth; ++j) {
            row[j] = i + "-" + j;
          }

          this.$set(this.scheme, i, row);
        }
      },
      
      onCellClick(i, j) {
        const pos = i+'-'+j;
        const newRow = this.scheme[i].slice(0);
        newRow[j] = this.scheme[i][j] === null ? pos : null;
        this.$set(this.scheme, i, newRow);

        setTimeout(this.updateDefinitions, 250);
        setTimeout(this.onDefinitionChange.bind(this, " ", pos, false), 500);
      },

      clearWord() {
        this.searchResults = null;
        this.pattern = "";
      },

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

        console.log("checked for " + counter + " words", availableWords);
        this.searchResults = availableWords;
      }
    },

    created() {
      this.dictionary = dictIta.dictionary;
      this.updateCruci();
      setTimeout( () => this.updateDefinitions(), 500 );
    },
  }
</script>

<style>
  .cell {
    height: 50px;
    width: 50px;
    margin: 1px;
    background: #ccc;
    position: relative;
  }
  .cell-black {
    background: #333;
  }
  .cell-definition-vertical {
    background: green;
  }
  .cell-definition-horizontal {
    background: salmon;
  }
  .cell-definition-vertical.cell-definition-horizontal {
    background: cyan;
  }

  .cell-definition {
    position: absolute;
    top: 1px;
    left: 3px;
  }

  .cell-text {
    position: absolute;
    font-size: 24px;
    top: 6px;
    left: 18px;
  }

</style>
