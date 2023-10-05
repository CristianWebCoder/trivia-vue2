<template>
    <div class="content">
        <!-- Barra de tiempo (esto es un placeholder, deberías implementar una barra de tiempo real si es necesario) -->
        <div class="number_and_progress">
            <span class="number">{{ currentQuestionNumber }}</span>
            <div class="bar_progress">
                <div class="progress" :style="{ width: progressPercentage + '%' }"></div>
            </div>
        </div>
        <span class="category">{{ question.category }}</span>
        <!-- Título de la Pregunta -->
        <h3 class="question" v-html="question.question"></h3>

        <!-- Opciones -->
        <div class="options">
            <button 
                v-for="option in allOptions" 
                :key="option"
                @click="selectAnswer(option)"
                v-html="option">
            </button>
        </div>
    </div>
</template>
  
<script>
    export default {
        props: ['question', 'currentQuestionNumber', 'progressPercentage'],
        computed: {
            allOptions() {
            // Combinamos las respuestas incorrectas y la correcta, y las mezclamos
            return this.shuffleArray([...this.question.incorrect_answers, this.question.correct_answer]);
        }
    },
    methods: {
        shuffleArray(array) {
            // Función para mezclar las opciones
            for (let i = array.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [array[i], array[j]] = [array[j], array[i]];
            }
            return array;
        },
        selectAnswer(answer) {
            // Emitimos el evento con la respuesta seleccionada
            this.$emit('answered', answer);
        }
    }
    };
</script>  