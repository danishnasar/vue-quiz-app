<template>
  <div>
    <ScoreBoard :winCount="this.winCount" :loseCount="this.loseCount"/>
    <template v-if="this.question">
      <h1 v-html="this.question"></h1>

      <template v-for="(answer, index) in this.answers" :key="index">
        <input
          :disabled="this.answerSubmitted"
          type="radio"
          name="options"
          :value="answer"
          v-model="this.chosenAnswer"
        />
        <label v-html="answer"></label><br />
      </template>

      <button
        v-if="!this.answerSubmitted"
        @click="this.submitedAnswer()"
        class="send"
        type="button"
      >
        Send
      </button>

      <section class="result" v-if="this.answerSubmitted">
        <h4
          v-if="this.chosenAnswer === this.correctAnswer"
          v-html="
            '&#9989; Congratulations, the answer' +
            this.chosenAnswer +
            ' is correct.'
          "
        ></h4>
        <h4
          v-else
          v-html="
            '&#10060; I`m sorry, you picked the wrong answer. The correct answer is ' +
            this.correctAnswer +
            '.'
          "
        ></h4>
        <button @click="getNewQuestion()" class="send" type="button">
          Next question
        </button>
      </section>
    </template>
  </div>
</template>

<script>
import ScoreBoard from './components/ScoreBoard.vue';
export default {
  name: "App",
  components: {ScoreBoard},

  data() {
    return {
      question: undefined,
      correctAnswer: undefined,
      incorrectAnswers: undefined,
      chosenAnswer: undefined,
      answerSubmitted: false,
      winCount: this.getScore('winCount'),
      loseCount: this.getScore('loseCount')
    };
  },

  computed: {
    answers() {
      let answers = [...this.incorrectAnswers];
      answers.splice(
        Math.round(Math.random() * (answers.length + 1)),
        0,
        this.correctAnswer
      );

      return answers;
    },
  },
  methods: {
    delay(duration) {
      return new Promise((resolve) => setTimeout(resolve, duration));
    },
    getScore(type) {
      const scoreData = localStorage.getItem('score');
      if (scoreData) {
        const scoreObj = JSON.parse(scoreData);
        if (type === 'winCount') return scoreObj.winCount;
        else return scoreObj.loseCount;
      } else {
        return 0;
      }
    },
    storeCache() {
      const scoreObj = { winCount: this.winCount, loseCount: this.loseCount };
      const scoreStr = JSON.stringify(scoreObj);
      localStorage.setItem('score', scoreStr);
    },
    submitedAnswer() {
      if (!this.chosenAnswer) alert("Please pick one of the options");
      else {
        this.answerSubmitted = true;
        if (this.correctAnswer === this.chosenAnswer) {
          this.winCount++;
          this.storeCache();
        }
        else {
          this.loseCount++;
          this.storeCache();
        }
      }
    },
    async getNewQuestion() {
      try {
        this.answerSubmitted = false;
        this.question = undefined;
        this.chosenAnswer = undefined;

        const api = "https://opentdb.com/api.php?amount=1";
        await this.delay(2500);
        await this.axios.get(api).then((response) => {
          this.question = response.data.results[0].question;
          this.correctAnswer = response.data.results[0].correct_answer;
          this.incorrectAnswers = response.data.results[0].incorrect_answers;
        });
      } catch (error) {
        alert(error);
      }
    },
  },

  created() {
    this.getNewQuestion();
  },
};
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin: 60px auto;
  max-width: 960px;

  input[type="radio"] {
    margin: 12px 4px;
  }
  button.send {
    margin-top: 12px;
    height: 40px;
    min-width: 120px;
    padding: 0 16px;
    color: #fff;
    background-color: #1867c0;
    border: 1px solid #1867c0;
    cursor: pointer;
  }
}
</style>
