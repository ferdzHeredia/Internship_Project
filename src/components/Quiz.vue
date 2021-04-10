<template>
  
  <div id="quiz-container">
    <div class = "navv">
    <ul>
      <li><a href="#home">Home</a></li>
      <li><a href="#news">Course</a></li>
      <li><a href="#contact">Take Quiz</a></li>
      <li><a href="#about">About</a></li>
    </ul>
    </div>
  <div id = "navdesign">

  </div>
    <div class="column" style="background-color:black;">     
    <a href="#Back"><h2>Back</h2></a>
    <p id= "algebra">Algebra Course</p>
    <!-- <button id = "takeQuizes"> Take Quiz 1</button>
    <button id = "takeQuizes"> Take Quiz 2</button> -->
    
    <button id= "takeQuizes" v-on:click="fetchQuestions(0)"
      >Take Quiz 1</button>
      <button id= "takeQuizes" v-on:click="fetchQuestions(1)"
      >Take Quiz 2</button> 
    </div>
    


    <div class = "multipleChoice-sidebar">
      <div class="correctAnswers">
      You have
      <strong>{{ correctAnswers }} correct {{ pluralizeAnswer }}!</strong>
    </div>
    <div class="correctAnswers">
      Currently at question {{ index + 1 }} of {{ questions.length }}  
    </div>
    <hr class="divider" />
    
    <form v-if="currentQuestion">
        <button
          v-for="answer in currentQuestion.answers"
          :index="currentQuestion.key"
          :key="answer"
          v-html="answer"
         @click.prevent="handleButtonClick"
         @click="showPopup = true"
        ></button>
      </form>

    <hr class="divider" />
    <button id= "cont" v-on:click="next()"
      v-show="showPopup"
      >continue</button>
      <button id="showExplanationButton"  
    v-show="showPopup"
    v-on:click="isHidden = !isHidden" >{{ isHidden ? 'Show Explanation' : 'Hide Explanation' }} </button>
    </div>
   
         
  <!-- Other elements here -->
<div class = "question">
  
    <img v-bind:src="loading ? ' ' : currentQuestion.question" v-bind:alt="pic">

    <!-- <h1 v-html="loading ? ' ' : currentQuestion.question"></h1> -->
           
              
</div>
 
  <div v-if ="!isHidden" id="explanation">
    <h1>Explanation goes Here!</h1>
    <img v-bind:src="loading ? ' ' : currentQuestion.solution" v-bind:alt="pic"> 
    <img v-bind:src="loading ? ' ' : currentQuestion.videoId" v-bind:alt="url"> 
    
     <!-- {{this.question.solution}}  -->
 
    

  </div>   
  </div>
</template>

<script>
export default {
  name: "Quiz",
  
  data() {
    return {
        
      questions: [],
      loading: true,
      index: 0,
      //to show and hide the explanation
      isHidden: true,
      //nextQuestions: true
      isActive: true,
      showPopup: false,
     // solu: Object
      
   };
  },
  computed: {
    currentQuestion() {
      if (this.questions !== []) {
        return this.questions[this.index];
      }
      return null;
    },
    score() {
      if (this.questions !== []) {
        // Here, we want to collect data in an object about the users statistics - later be emitted on an event when users finishes quiz
        return {
          allQuestions: this.questions.length,
          answeredQuestions: this.questions.reduce((count, currentQuestion) => {
            if (currentQuestion.userAnswer) {
              // userAnswer is set when user has answered a question, no matter if right or wrong
              count++;
            }
            return count;
          }, 0),
          correctlyAnsweredQuestions: this.questions.reduce(
            (count, currentQuestion) => {
              if (currentQuestion.rightAnswer) {
                // rightAnswer is true, if user answered correctly
                count++;
              }
              return count;
            },
            0
          ),
        };
      } else {
        return {
          allQuestions: 0,
          answeredQuestions: 0,
          correctlyAnsweredQuestions: 0,
        };
      }
    },
    correctAnswers() {
      if (this.questions && this.questions.length > 0) {
        let streakCounter = 0;
        this.questions.forEach(function(question) {
          if (!question.rightAnswer) {
            return;
          } else if (question.rightAnswer === true) {
            streakCounter++;
          }
        });
        return streakCounter;
      } else {
        return "--";
      }
    },
    pluralizeAnswer() {
      // For grammatical correctness
      return this.correctAnswers === 1 ? "Answer" : "Answers";
    },
    quizCompleted() {
      if (this.questions.length === 0) {
        return false;
      }
      /* Check if all questions have been answered */
      let questionsAnswered = 0;
      this.questions.forEach(function(question) {
        question.rightAnswer !== null ? questionsAnswered++ : null;
      });
      return questionsAnswered === this.questions.length;
    },
  },
  watch: {
    quizCompleted(completed) {
      /*
       * Watcher on quizCompleted fires event "quiz-completed"
       * up to parent App.vue component when completed parameter
       * returned by quizCompleted computed property true
       */
      completed &&
        setTimeout(() => {
          this.$emit("quiz-completed", this.score);
        }, 3000); // wait 3 seconds until button animation is over
    },
  },
  methods: {
    async fetchQuestions(quizId) {

       let url = "http://localhost:3333/questions/";

      this.loading = true;
      let response = await fetch(       
        url
      //  "http://localhost:3333/questions" 
      );
      

      let jsonResponse = [];
      jsonResponse = await response.json();
      let index = 0; // index is used to identify single answer
      
      
      let data = jsonResponse[quizId].results.map((question) => {
      // Solutions.Solution = [question.videoId, question.Solution];
        question.answers = [          
          question.correct_answer,
          ...question.incorrect_answers,
        ];
      //console.log(question.solution)
      
        
        /* Shuffle question.answers array */
        for (let i = question.answers.length - 1; i > 0; i--) {
          const j = Math.floor(Math.random() * (i + 1));
          [question.answers[i], question.answers[j]] = [
            question.answers[j],
            question.answers[i],
          ];
        }
        // mention in Step 1
       
        question.rightAnswer = null;
        question.key = index;
        index++;
        return question;
      });
      this.solu = jsonResponse[quizId].results.solution
      
      this.questions = data;
      this.loading = false;
    },
    handleButtonClick: function(event) {
      /* Find index to identify question object in data */
      let index = event.target.getAttribute("index");
      let pollutedUserAnswer = event.target.innerHTML; // innerHTML is polluted with decoded HTML entities e.g ' from &#039;
      /* Clear from pollution with ' */
      let userAnswer = pollutedUserAnswer.replace(/'/, "&#039;");

      /* Set userAnswer on question object in data */
      this.questions[index].userAnswer = userAnswer;

      /* Set class "clicked" on button with userAnswer -> for CSS Styles; Disable other sibling buttons */
      event.target.classList.add("clicked");
      let allButtons = document.querySelectorAll(`[index="${index}"]`);

      for (let i = 0; i < allButtons.length; i++) {
        if (allButtons[i] === event.target) continue;

        allButtons[i].setAttribute("disable", "");
      }
      //  for (let i = 0; i < allButtons.length; i++) {
      //   if (allButtons[i] === event.target) continue;

      //   this.results[i].setAttribute("enable", "");
      // }
       

      /* Invoke checkAnswer to check Answer */
       this.checkAnswer(event, index);
      
    },  

    checkAnswer: function(event, index) {
      let question = this.questions[index];
      if (question.userAnswer) {
        // if (this.index < this.questions.length - 1) {
        //   setTimeout(
        //     function() {
        //       this.index += 1;
        //     }.bind(this),
        //     3000
        //   );
        // }
        if (question.userAnswer === question.correct_answer) {
          /* Set class on Button if user answered right, to celebrate right answer with animation joyfulButton */
          event.target.classList.add("rightAnswer");
          /* Set rightAnswer on question to true, computed property can track a streak out of 10 questions */
          this.questions[index].rightAnswer = true;
          //question.correct_answer
         
          // this.questions[index].correct_answer = null;
          
        } else {
          /* Mark users answer as wrong answer */
          event.target.classList.add("wrongAnswer");
          this.questions[index].rightAnswer = false;
          /* Show right Answer */
          let correctAnswer = this.questions[index].correct_answer;
          let allButtons = document.querySelectorAll(`[index="${index}"]`);
          allButtons.forEach(function(button) {
            if (button.innerHTML === correctAnswer) {
              button.classList.add("showRightAnswer");
            }
          });
        }
      

      }
    },
    el: '#cont',
   next: function () {
      
        if (this.index < this.questions.length - 1) {
          setTimeout(
            function() {
              this.index += 1;
              this.showPopup = false
            }.bind(this),
            500,
          );
        }
      
    },
     explanations() {

       console.log(this.question.solution)
    //return this.question.solution
      //  return this.solu

    //   let a = this.jsonResponse[0].results
    //   return a
    //  // return jsonResponse[0].results.solution
     },
  },
  mounted() {
    this.fetchQuestions();
  },

}
</script>

<style scoped>
#quiz-container {
  /* margin: 1rem auto; */
  width:100%;
  position: relative;
}

#logo-headline {
  font-size: 3rem;
  padding: 0.5rem;
  color: #f50057;
  text-align: center;
}

#logo-crown {
  display: block;
  width: 40%;
  margin: 0 auto;
}

@media only screen and (max-width: 500px) {
  #logo-crown {
    width: 30%;
  }

  #logo-headline {
    font-size: 1.8rem;
  }
}

h1 {
  font-size: 1.3rem;
  padding: 0.7rem;
}

.divider {
  margin: 0.5rem 0;
  border: 3px solid rgba(102, 255, 166, 0.7);
  border-radius: 2px;
  box-shadow: 3px 5px 5px rgba(0, 0, 0, 0.3);
}

form {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: center;
}

button {
  font-size: 1.1rem;
  box-sizing: border-box;
  padding: 1rem;
  margin: 0.3rem;
  width: 40%;
  background-color: rgba(100, 100, 100, 0.3);
  border: none;
  border-radius: 0.4rem;
  box-shadow: 3px 5px 5px rgba(0, 0, 0, 0.2);
}

button:hover:enabled {
  transform: scale(1.02);
  box-shadow: 0 3px 3px 0 rgba(0, 0, 0, 0.14), 0 1px 7px 0 rgba(0, 0, 0, 0.12),
    0 3px 1px -1px rgba(0, 0, 0, 0.2);
}

button:focus {
  outline: none;
}

button:active:enabled {
  transform: scale(1.05);
}

@keyframes flashButton {
  0% {
    opacity: 1;
    transform: scale(1.01);
  }
  50% {
    opacity: 0.7;
    transform: scale(1.02);
  }
  100% {
    opacity: 1;
    transform: scale(1);
  }
}

button.clicked {
  pointer-events: none;
}

button.rightAnswer {
  animation: flashButton;
  animation-duration: 700ms;
  animation-delay: 200ms;
  animation-iteration-count: 3;
  animation-timing-function: ease-in-out;
  color: black;
  background: linear-gradient(
    210deg,
    rgba(0, 178, 72, 0.25),
    rgba(0, 178, 72, 0.5)
  );
}

button.wrongAnswer {
  color: black;
  background: linear-gradient(
    210deg,
    rgba(245, 0, 87, 0.25),
    rgba(245, 0, 87, 0.5)
  );
}

button.showRightAnswer {
  animation: flashButton;
  animation-duration: 700ms;
  animation-delay: 200ms;
  animation-iteration-count: 2;
  animation-timing-function: ease-in-out;
  color: black;
  background: linear-gradient(
    210deg,
    rgba(0, 178, 72, 0.25),
    rgba(0, 178, 72, 0.5)
  );
}

.correctAnswers { 
 text-align: center;
 padding-bottom: 2%;
}

/* Multiple choice side bar*/
.multipleChoice-sidebar{
  margin-right: 15px;
  position: absolute;;
  padding-top: 5em;
  right: 0;
  text-align: left;
  float: left;
  z-index:1;
  padding: 100px 32px 88px 48px; 
  width: 40%;

  
}
.question{
   float: left; 
  width: 45%;
  position: relative;
  margin-top: 2%;
  padding-bottom: 5%;
  margin-right: 20%;
    
}

#cont{
  width: 50%;
  position: center;
  color: black;
  background-image: url('/mintNav.jpg');
  border-style: solid;
  border-color: white;
  margin-left: 25%;
  
  
}
#showExplanationButton{
  width: 50%;
  position: center;
  color: rgb(10, 10, 10);
  background-image: url('/mintNav.jpg');
  margin-left: 25%;
  border-style: solid;
  border-color: white;
  
  
}

 ul {
  background-image: url('/mintNav.jpg');
  list-style-type: none;
  margin: 0;
  padding: 0;
  overflow: hidden;
  /* position: fixed; */
  top: 0;
  width:100%

}

li {
  float: left;
}

li a {
  display: block;
  color: rgb(14, 13, 13);
  text-align: center;
  padding: 14px 16px;
  text-decoration: none;
}

li a:hover:not(.active) {
  background-color: rgb(241, 236, 236);
}

.active {
  background-color: #4CAF50;
}

#explanation {
  margin-bottom: 5em;
  position:relative;
  width:50%;
  margin-left: 340px;
  padding-right: 150px;

}
#hidee{
position: fixed;
     bottom: 0;
     right: 0;
}
.correctAnswers{
  padding-bottom: 5%;
}
.math-img{
  padding-top: 1%;
  padding-left: 2%;
}
/* Create two equal columns that floats next to each other */
.column {
  /* background-image: url('/images-gradient.jpg'); */
  text-align: right;
  color: black;
  float: left;
  width: 18%;
  padding: 10px;
  height: 710px; /* Should be removed. Only for demonstration */

  
}
.column a #algebra{
  line-height: 5em;
  text-decoration: none;
  color: white;
}
.column a:hover:enabled {
  transform: scale(1.02);
  box-shadow: 0 3px 3px 0 rgba(0, 0, 0, 0.14), 0 1px 7px 0 rgba(0, 0, 0, 0.12),
    0 3px 1px -1px rgba(0, 0, 0, 0.2);
}

.column a:focus {
  outline: none;
}

.column a:active:enabled {
  transform: scale(1.05);
}
#navdesign{

  background-image: url('/Gradient_Background.jpg'); 
  list-style-type: none;
  margin: 0;
  padding: 0;
  overflow: hidden;
  height: 4em;
  /* position: fixed; */
  top: 0;
  width:100%
}
#takeQuizes{
  margin-top: 5%;;
  float: right;
  padding-top: 5px;
  padding-bottom: 5px;
  background-color: white;
  width: 50%;
}
</style>


    
   
    
    