<template>
    <div class="game">
        <!-- Diseño Inicial -->
        <div class="score">
            <span>userName: {{ score }}pts</span>
        </div>
        <div class="first_view" v-if="!started && !finished">
            <h1>TRIVIA GAME</h1>
            <div class="custom_select">
                <select v-model="selectedCategory">
                    <option v-for="category in categories" :key="category.id" :value="category.id">
                        {{ category.name }}
                    </option>
                </select>
            </div>
            <div class="custom_select">
                <select v-model="selectedDifficulty">
                    <option value="easy">Fácil</option>
                    <option value="medium">Media</option>
                    <option value="hard">Difícil</option>
                </select>
            </div>
            <div class="custom_submit">
                <button @click="startTrivia">Empezar</button>
            </div>
        </div>
    
        <!-- Vista de Pregunta -->
        <transition name="fade">
            <div class="second_view" v-if="started && !finished">
                <TriviaQuestion :question="currentQuestion" :currentQuestionNumber="currentQuestionIndex + 1" :progressPercentage="progressPercentage" @answered="handleAnswer"/>
            </div>
        </transition>
    
        <!-- Vista de Resultado -->
        <div class="third_view" v-if="finished">
            <img :src="TrofeoPath" alt="Trofeo">
            <h2>Felicitaciones, hiciste {{ score }} puntos.</h2>
            <button class="button_game" @click="restartTrivia">Volver a jugar</button>
            <button class="button_view_questions" @click="showAnswers = !showAnswers">Ver respuestas correctas</button>
            
            <table class="questions_and_answers" v-if="showAnswers">
                <thead>
                    <tr>
                        <th>Preguntas contestadas</th>
                        <th>Respuestas correctas</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(q, index) in questions" :key="index">
                        <td>
                            <span v-if="userAnswers[index] === q.correct_answer">✅</span>
                            <span v-else>❌</span>
                            <span v-html="q.question"></span>
                        </td>
                        <td>
                            <span class="correct_answer" v-html="q.correct_answer"></span>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<script>
    import TriviaQuestion from './TriviaQuestion.vue';
    import axios from 'axios';
    
    export default {
        components: {
            TriviaQuestion
        },
        data() {
            return {
                currentQuestionNumber: 1,
                categories: [],
                selectedCategory: null,
                selectedDifficulty: 'easy',
                started: false,
                finished: false,
                currentQuestion: {},
                questions: [],
                score: 0,
                currentQuestionIndex: 0,
                TrofeoPath: require('@/assets/trofeo.png'),
                userAnswers: [],
                showAnswers: false
            };
        },
        computed: {
            progressPercentage() {
                return ((this.currentQuestionIndex + 1) / this.questions.length) * 100;
            }
        },
        async mounted() {
            try {
                const response = await axios.get('https://opentdb.com/api_category.php');
                this.categories = response.data.trivia_categories;
                if (this.categories.length > 0) {
                this.selectedCategory = this.categories[0].id;
                }
            } catch (error) {
                console.error('Error fetching categories: ', error);
            }
        },
        methods: {
            async startTrivia() {
                try {
                const response = await axios.get(`https://opentdb.com/api.php?amount=10&category=${this.selectedCategory}&difficulty=${this.selectedDifficulty}`);
                this.questions = response.data.results;
                this.currentQuestion = this.questions[this.currentQuestionIndex];
                this.started = true;
                } catch (error) {
                console.error('Error fetching questions: ', error);
                }
            },
            handleAnswer(answer) {
                // Añadir la respuesta al array userAnswers:
                this.userAnswers.push(answer);

                if (answer === this.currentQuestion.correct_answer) {
                    this.score += 1;
                }

                this.currentQuestionIndex += 1;

                if (this.currentQuestionIndex < this.questions.length) {
                    this.currentQuestion = this.questions[this.currentQuestionIndex];
                } else {
                    this.finished = true;
                    this.started = false;
                }
            },
            restartTrivia() {
                this.started = false;
                this.finished = false;
                this.score = 0;
                this.currentQuestionIndex = 0;
                this.currentQuestion = {};
                this.questions = [];
            },
        },
    };
</script>  