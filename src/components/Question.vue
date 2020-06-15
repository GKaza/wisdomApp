<template>
  <div id="question-container">
    <h3 class="question">{{question}}</h3>
    <button
      v-for="answer in mixAnswers"
      :key="answer"
      class="btn"
      @click="answered(answer)"
    >{{answer}}</button>
  </div>
</template>

<script>
export default {
  name: "Question",
  props: ["question", "correct", "incorrect"],
  methods: {
    answered(val) {
      this.$emit("answered", this.isCorrect(val));
    },
    isCorrect(ans) {
      return ans === this.correct;
    }
  },
  computed: {
    mixAnswers() {
      let random = this.incorrect.slice();
      random.splice(Math.floor(Math.random() * 4), 0, this.correct);
      return random;
    }
  }
};
</script>

<style scoped>
</style>
