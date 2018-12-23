<template>
  <div class="container">
    <div class="container">
      <h1>Wordscapes Word Finder</h1>
      <h2>Find all the words, maximize your coinage!</h2>
      <p>{{ wordCount }}</p>
      <hr>
      <div class="row">
        <div class="col-sm-2">
          <div v-show="possiblePoints() > 0" style="text-align: center;">
            <span>Possible Points</span>
            <h1 style="font-size: 60pt;color: rgb(51, 121, 183); margin: 0;">{{ possiblePoints() }}</h1>
          </div>
        </div>
        <div class="form-horizontal col-sm-8">
          <div class="form-group">
            <label class="control-label col-sm-6">Enter your letters.</label>
            <div class="col-sm-6">
              <input
                id="lettersInput"
                type="text"
                class="form-control"
                v-model="letters"
                style="text-transform: uppercase;"
              >
              <span style="font-style: italic;">{{ lettersCount }} letter(s)</span>
            </div>
          </div>

          <div class="form-group">
            <label class="control-label col-sm-6">What is the minimum length of the word?</label>
            <div class="col-sm-6">
              <div class="btn-group" role="group" aria-label="...">
                <button
                  type="button"
                  class="btn btn-default"
                  :class="{active: minLength == 3}"
                  @click.prevent="minLength = 3"
                >3</button>
                <button
                  type="button"
                  class="btn btn-default"
                  :class="{active: minLength == 4}"
                  @click.prevent="minLength = 4"
                >4</button>
              </div>
            </div>
          </div>

          <div class="col-sm-offset-6 col-sm-6">
            <button class="btn btn-primary" @click.prevent="search">Search</button>
          </div>
        </div>
      </div>
      <hr>
      <div v-for="wordList in results" :key="wordList.LengthOfWord">
        <h3>{{ wordList.LengthOfWord }} Letter(s)</h3>
        <hr>
        <div class="row">
          <div
            class="col-lg-2 col-md-3 col-sm-4 col-xs-6"
            v-for="word in wordList.Words"
            :key="word"
          >
            <div style="margin-bottom: 1em;">
              <button
                class="btn btn-xs"
                :class="{ 'btn-danger' : !isWordDeleted(word), 'btn-primary' : isWordDeleted(word) }"
                @click.prevent="removeWord(wordList.LengthOfWord, word)"
              >
                <i class="fa fa-trash" aria-hidden="true" v-show="!isWordDeleted(word)"></i>
                <i class="fa fa-undo" aria-hidden="true" v-show="isWordDeleted(word)"></i>
              </button>
              &nbsp;
              <span
                style="cursor: pointer;"
                @click="removeWord(wordList.LengthOfWord, word)"
                :class="{ 'deleted-word' : isWordDeleted(word) }"
              >{{ word }}</span>
            </div>
          </div>
        </div>
      </div>
      <div class="row">
        <hr>
        <p>
          <button
            class="btn btn-primary"
            @click.prevent="updateDictionary"
            :disabled="!canUpdateDictionary"
          >Update Dictionary</button>
          &nbsp;
          <button class="btn btn-default" @click.prevent="reset">Reset</button>
        </p>
        <div class="row">
          <div class="col-sm-6">
            <div class="well">
              <h4>Words to Remove</h4>
              <div v-for="word in wordsToRemove" :key="word">{{ word }}</div>
            </div>
          </div>
          <div class="col-sm-6">
            <div class="well">
              <h4>Words to Add</h4>
              <textarea
                id="WordsToAddTextbox"
                class="form-control"
                rows="10"
                v-model="wordsToAdd"
                style="text-transform: uppercase;"
              />
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// production:
let api = "http://mythic/wordscapes/";

// dev:
// let api = "http://localhost:62658/";

import axios from "axios";
import accounting from "accounting";
import _ from "underscore";
import notie from "notie";

export default {
  name: "AppComponent",

  data() {
    return {
      letters: "",
      minLength: 3,
      results: [],
      wordsToRemove: [],
      wordsToAdd: "",
      dictionaryWordCount: -1
    };
  },

  created() {
    this.loadWordCount();
  },

  computed: {
    canUpdateDictionary() {
      return this.wordsToRemove.length > 0 || this.wordsToAdd.length > 0;
    },

    lettersCount() {
      return this.letters.length;
    },

    wordCount() {
      if (this.dictionaryWordCount >= 0) {
        return (
          "Our custom dictionary contains " +
          accounting.formatNumber(this.dictionaryWordCount) +
          " words!"
        );
      } else {
        return "";
      }
    }
  },

  methods: {
    isWordDeleted(word) {
      return _.contains(this.wordsToRemove, word);
    },

    loadWordCount() {
      axios
        .get(api + "words/init")
        .then(resp => {
          if (resp.data.WordCount) {
            this.dictionaryWordCount = resp.data.WordCount;
          }
        })
        .catch(() => {
          this.dictionaryWordCount = -1;
        });
    },

    possiblePoints() {
      let points = 0;

      _.each(this.results, item => {
        points += item.Words.length;
      });

      points -= this.wordsToRemove.length;

      let wordsToAddArray = _.filter(this.wordsToAdd.split("\n"), word => {
        return word.length > 0;
      });

      points += wordsToAddArray.length;

      return points;
    },

    search() {
      axios
        .get(
          api +
            "words/Suggestions?Letters=" +
            this.letters +
            "&MinimumWordLength=" +
            this.minLength
        )
        .then(resp => {
          if (resp.data.WordLists) {
            this.results = resp.data.WordLists;
          }
        });
    },

    removeWord(lengthOfWord, word) {
      if (_.contains(this.wordsToRemove, word)) {
        this.wordsToRemove = _.filter(this.wordsToRemove, w => {
          return w !== word;
        });
      } else {
        this.wordsToRemove.push(word);
        this.wordsToRemove = _.sortBy(this.wordsToRemove, word => {
          return word;
        });
      }
    },

    reset() {
      this.letters = "";
      this.minLength = 3;
      this.results = [];
      this.wordsToRemove = [];
      this.wordsToAdd = "";
      this.loadWordCount();

      document.getElementById("lettersInput").focus();
    },

    updateDictionary() {
      axios
        .post(api + "words/UpdateDictionary", {
          wordsToRemove: this.wordsToRemove,
          wordsToAdd: this.wordsToAdd.split("\n")
        })
        .then(resp => {
          if (resp.data.Err === "") {
            notie.alert({
              type: "success",
              text: "The dictionary has been updated."
            });

            this.reset();
          } else {
            notie.alert({
              type: "error",
              text: resp.data.Err
            });
          }
        });
    }
  }
};
</script>

<style scoped>
.deleted-word {
  text-decoration: line-through;
}
</style>
