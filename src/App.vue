<template>
  <div id="app">
    <h1 id="header">Wisdom Trivia Game</h1>
    <form @submit.prevent="formSubmit" v-if="this.show.form" class="form-container">
      <div class="form-item">
        <label for="name">Player Name:</label>
        <input id="name" v-model="player.name" />
      </div>
      <div class="form-item">
        <label for="difficulty">Difficulty:</label>
        <select id="difficulty" v-model="player.difficulty">
          <option value="easy">Easy</option>
          <option value="medium">Medium</option>
          <option value="hard">Hard</option>
        </select>
      </div>
      <input type="submit" class="btn" />
      <p v-if="this.show.error" class="error-message">Please choose your name and difficulty.</p>
    </form>
    <div v-if="this.show.question">
      <Question
        v-for="(question,index) in questions"
        v-show="questionOrder(index)"
        :key="index"
        :id="index"
        :question="question.question"
        :correct="question.correct_answer"
        :incorrect="question.incorrect_answers"
        @answered="answered($event)"
      />
    </div>
    <div v-if="this.show.results">
      <h1 class="results">Game Over!</h1>
      <h2 class="results">Thank you for playing {{player.name}}.</h2>
      <h2 class="results">You answered {{player.score}} out of 10 correctly!</h2>
      <button class="btn results" @click="seeHighscores">See highscores</button>
    </div>
    <div v-if="this.show.highscores">
      <h2>{{player.difficulty}}</h2>
      <ol>
        <li
          v-for="(item,index) in highscores[this.player.difficulty]"
          :key="index"
        >{{item.name}} - {{item.score}}</li>
      </ol>
    </div>
  </div>
</template>

<script>
import Question from "./components/Question.vue";

export default {
  name: "App",
  components: {
    Question
  },
  data() {
    return {
      counter: 0,
      show: {
        form: true,
        question: false,
        error: false,
        results: false,
        highscores: false
      },
      player: {
        name: null,
        difficulty: null,
        score: 0
      },
      highscores: {
        easy: [
          {
            name: "john",
            difficulty: "easy",
            score: 5
          },
          {
            name: "joan",
            difficulty: "easy",
            score: 6
          }
        ],
        medium: [],
        hard: []
      },
      questions: null
    };
  },
  methods: {
    async formSubmit() {
      if (this.player.name && this.player.difficulty) {
        switch (this.player.difficulty) {
          case "hard":
            this.questions = await fetch(
              "https://opentdb.com/api.php?amount=10&difficulty=hard&type=multiple"
            )
              .then(response => response.json())
              .then(data => data.results);
            break;
          case "medium":
            this.questions = await fetch(
              "https://opentdb.com/api.php?amount=10&difficulty=medium&type=multiple"
            )
              .then(response => response.json())
              .then(data => data.results);
            break;
          default:
            this.questions = await fetch(
              "https://opentdb.com/api.php?amount=10&difficulty=easy&type=multiple"
            )
              .then(response => response.json())
              .then(data => data.results);
        }
        this.show.form = false;
        this.show.question = true;
      } else {
        this.show.error = true;
      }
    },
    questionOrder(index) {
      return index === this.counter;
    },
    answered(ans) {
      this.counter++;
      if (ans) {
        this.player.score++;
      }
      if (this.counter === 10) {
        this.show.question = false;
        this.show.results = true;
      }
    },
    seeHighscores() {
      this.show.results = false;
      this.show.highscores = true;
    }
  },
  computed: {}
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin: 0px;
  height: 100%;
  background: rgb(255, 201, 234);
  background: linear-gradient(
    302deg,
    rgba(255, 201, 234, 1) 0%,
    rgba(181, 242, 255, 1) 100%
  );
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
}

#header {
  padding: 40px;
}

.form-item {
  padding: 20px;
}

.btn {
  color: rgb(255, 255, 255);
  background-color: #2c3e50;
  cursor: pointer;
  padding: 1%;
  border-radius: 30px;
  border: 0px;
  margin: 10px;
}

.btn:hover {
  color: #2c3e50;
  background-color: #ffe6f5;
}

.error-message {
  font-size: 0.7em;
  color: rgb(255, 0, 0);
}
</style>
