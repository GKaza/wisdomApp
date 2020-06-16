<template>
  <div id="app">
    <h1 class="header">
      <a @click="this.reload">Wisdom Trivia Game</a>
    </h1>
    <div v-if="this.show.form">
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
        <input type="submit" value="Start Game" class="btn" />
        <p v-if="this.show.error" class="error-message">Please choose your name and difficulty.</p>
      </form>
      <div class="loader"></div>
    </div>

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
      <button class="btn" @click="seeHighscores">See highscores</button>
    </div>
    <div v-if="this.show.highscores">
      <h2 class="header">{{player.difficulty}}</h2>
      <ol class="scores-container">
        <li
          v-for="(item,index) in highscores[this.player.difficulty]"
          :key="index"
          class="scores"
        >{{item.name}} - {{item.score}}/10</li>
      </ol>
      <button class="delete" @click="deleteHighscores">Delete all highscores</button>
      <div>
        <button class="btn" @click="reload">Play Again!</button>
      </div>
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
      highscores: JSON.parse(localStorage.getItem("highscores")),
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
    reload() {
      window.location.reload();
    },
    answered(ans) {
      this.counter++;
      if (ans) {
        this.player.score++;
      }
      if (this.counter === 10) {
        this.highscores[this.player.difficulty].push(this.player);
        this.highscores[this.player.difficulty].sort(function(a, b) {
          return b.score - a.score;
        });
        localStorage.setItem("highscores", JSON.stringify(this.highscores));
        this.show.question = false;
        this.show.results = true;
      }
    },
    deleteHighscores() {
      localStorage.removeItem("highscores");
      this.reload();
    },
    seeHighscores() {
      this.show.results = false;
      this.show.highscores = true;
    }
  },
  computed: {},
  created: function() {
    if (!this.highscores) {
      this.highscores = {
        easy: [],
        medium: [],
        hard: []
      };
    }
  }
};
</script>

<style>
@import url("https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap");
#app {
  font-family: "Press Start 2P", cursive;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin: 0px;
  height: 100%;
}

.results {
  line-height: 3rem;
}

a {
  cursor: pointer;
}

.header {
  padding: 40px;
}

.form-container {
  min-width: fit-content;
  width: 300px;
  padding: 20px;
  margin: 40px auto;
  border: 2px solid #617b942c;
  border-radius: 50px;
}

.form-item {
  padding: 20px;
}

.loader {
  display: inline-block;
  margin: 40px;
  width: 80px;
  height: 80px;
  background-color: #86bbbd;
  border-radius: 0;
  animation-name: rotate;
  animation-duration: 3s;
  animation-iteration-count: infinite;
  animation-timing-function: ease-in-out;
  animation-direction: alternate;
}

@keyframes rotate {
  0% {
    transform: rotate(0deg);
    border-radius: 0;
  }
  25% {
    transform: rotate(45deg);
    border-radius: 0;
    border-top-left-radius: 50%;
    background-color: #76949f;
  }
  50% {
    transform: rotate(90deg);
    border-radius: 0;
    border-top-left-radius: 50%;
    border-top-right-radius: 50%;
    background-color: #6a6b83;
  }
  75% {
    transform: rotate(135deg);
    border-radius: 0;
    border-top-left-radius: 50%;
    border-top-right-radius: 50%;
    border-bottom-right-radius: 50%;
    background-color: #5f506b;
  }
  100% {
    transform: rotate(180deg);
    border-radius: 50%;
  }
}

.btn {
  color: rgb(255, 255, 255);
  background-color: #2c3e50;
  cursor: pointer;
  padding: 1%;
  border-radius: 30px;
  border: 0px;
  margin: 10px;
  outline: 0px;
}

.btn:hover {
  color: #2c3e50;
  background-color: #ffe6f5;
}

.delete {
  color: #2c3e50;
  background-color: #ffe6f5;
  cursor: pointer;
  border-radius: 30px;
  border: 0px;
  margin: 20px;
}

.delete:hover {
  color: #ff0000;
  background-color: #2c3e50;
}

.scores-container {
  margin: auto;
  min-width: 200px;
  width: fit-content;
}

.scores {
  padding: 8px;
}

.error-message {
  font-size: 0.7em;
  color: rgb(255, 0, 0);
}
</style>
