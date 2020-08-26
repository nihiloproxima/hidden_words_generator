<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <div class="d-flex align-center">Générateur de mots cachés</div>

      <v-spacer></v-spacer>
    </v-app-bar>

    <v-main>
      <v-form>
        <v-container>
          <v-row>
            <v-col cols="12">
              <v-row>
                <v-col cols="6">
                  <v-text-field v-model.number="wordsLength" label="Nombre de mots" type="number"></v-text-field>
                </v-col>
                <v-col cols="6">
                  <v-text-field v-model.number="size" label="Taille de la grille" type="number"></v-text-field>
                </v-col>
              </v-row>
              <v-select
                v-model="modes"
                :items="availableModes"
                attach
                chips
                label="Difficulté"
                multiple
              ></v-select>
            </v-col>
            <v-col cols="12">
              <h2>Liste des mots :</h2>
              <v-row>
                <v-col cols="4" v-for="index of wordsLength" :key="index">
                  <v-text-field v-model="words[index - 1]" clearable></v-text-field>
                </v-col>
              </v-row>
              <div class="text-center">
                <v-btn color="primary" @click="generate">Générer la grille</v-btn>
              </div>
            </v-col>
          </v-row>
        </v-container>
        <v-container>
          <p class="text-center" v-for="word of sortedWords" :key="word">{{ word}}</p>
        </v-container>
        <v-container>
          <table v-if="result.length">
            <tr v-for="(row, index) of result" :key="index">
              <td v-for="(value, colIndex) of row" :key="(colIndex)">{{value}}</td>
            </tr>
          </table>
        </v-container>
      </v-form>
    </v-main>
  </v-app>
</template>

<script>
import _ from "lodash";
import dictionnary from "./words.json";

export default {
  name: "App",
  data: () => ({
    availableModes: [
      "diagonal",
      "horizontal",
      "vertical",
      "horizontal inverse",
      "vertical inverse",
    ],
    modes: ["diagonal", "horizontal", "vertical"],
    wordsLength: 6,
    word: "",
    words: [],
    result: [],
    size: 15,
    sortedWords: [],
  }),
  components: {},
  methods: {
    randomWords() {
      return _.sampleSize(dictionnary, this.wordsLength);
    },

    addWord() {
      this.words.push(this.word);
    },
    generate() {
      this.result = [];

      if (this.words.length === 0) {
        this.words = this.randomWords();
      }

      this.words = _.shuffle(
        this.words.filter((word) => !_.isUndefined(word))
      ).map((word) => _.toUpper(word));

      const longerWord = this.words.sort((a, b) => b.length - a.length)[0];
      const size =
        this.size < longerWord.length ? longerWord.length + 1 : this.size;

      this.generateGrid(size);

      for (let word of this.words) {
        this.tryPlacing(size, word);
      }

      this.cleanGrid();
      this.sortedWords = this.words.sort();
    },

    tryPlacing(size, word) {
      let direction = _.sample(this.modes);

      let x = _.random(size - 1);
      let y = _.random(size - 1);

      if (direction === "diagonal") {
        while (!this.canFitDiagonal(x, y, size, word)) {
          x = _.random(size - 1);
          y = _.random(size - 1);
        }
        return this.addDiagonalWord(x, y, word);
      } else if (direction === "horizontal") {
        while (!this.canFitHorizontal(x, y, size, word)) {
          x = _.random(size - 1);
          y = _.random(size - 1);
        }
        return this.addHorizontal(x, y, word);
      } else if (direction === "vertical") {
        while (!this.canFitVertical(x, y, size, word)) {
          x = _.random(size - 1);
          y = _.random(size - 1);
        }
        return this.addVertical(x, y, word);
      } else if (direction === "vertical inverse") {
        while (
          !this.canFitVertical(x, y, size, word.split("").reverse().join(""))
        ) {
          x = _.random(size - 1);
          y = _.random(size - 1);
        }
        return this.addVertical(x, y, word.split("").reverse().join(""));
      } else if (direction === "horizontal inverse") {
        while (
          !this.canFitHorizontal(x, y, size, word.split("").reverse().join(""))
        ) {
          x = _.random(size - 1);
          y = _.random(size - 1);
        }
        return this.addHorizontal(x, y, word.split("").reverse().join(""));
      }
      return false;
    },

    addDiagonalWord(x, y, word) {
      for (let i = 0; i < word.length; i++) {
        this.result[y + i][x + i] = word[i];
      }
      return true;
    },

    canFitDiagonal(x, y, size, word) {
      for (let i = 0; i < word.length; i++) {
        if (
          x + i >= size ||
          y + i >= size ||
          ![0, word[i]].includes(this.result[y + i][x + i]) ||
          this.result[y + i] === undefined ||
          (this.result[y + i] && this.result[y + i][x + i] === undefined)
        ) {
          return false;
        }
      }
      return true;
    },

    addHorizontal(x, y, word) {
      for (let i = 0; i < word.length; i++) {
        this.result[y][x + i] = word[i];
      }
      return true;
    },

    canFitHorizontal(x, y, size, word) {
      for (let i = 0; i < word.length; i++) {
        if (x + i >= size || ![0, word[i]].includes(this.result[y][x + i])) {
          return false;
        }
      }
      return true;
    },

    addVertical(x, y, word) {
      for (let i = 0; i < word.length; i++) {
        this.result[y + i][x] = word[i];
      }
      return true;
    },

    canFitVertical(x, y, size, word) {
      for (let i = 0; i < word.length; i++) {
        if (y + i >= size || ![0, word[i]].includes(this.result[y + i][x])) {
          return false;
        }
      }
      return true;
    },

    generateGrid(size) {
      for (let i = 0; i < size; i++) {
        this.result[i] = [];
        for (let j = 0; j < size; j++) {
          this.result[i].push(0);
        }
      }
    },

    cleanGrid() {
      const alphabet = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
      for (let i = 0; i < this.result.length; i++) {
        for (let j = 0; j < this.result.length; j++) {
          if (this.result[i][j] === 0) {
            this.result[i][j] = _.sample(alphabet);
          }
        }
      }
    },
  },
};
</script>

<style scoped>
table {
  margin: 0 auto;
  border: solid 1px black;
  padding: 5px;
}
td {
  padding: 10px;
  width: 50px;
}
</style>
