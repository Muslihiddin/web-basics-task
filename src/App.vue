<script setup lang="ts">
import { useId, ref, computed } from "vue";
import { Button } from "@/components/ui/button";
import { Label } from "./components/ui/label";
import { RadioGroup, RadioGroupItem } from "./components/ui/radio-group";
import {
  Card,
  CardContent,
  CardHeader,
  CardTitle,
  CardDescription,
} from "@/components/ui/card";
import {
  Stepper,
  StepperItem,
  StepperSeparator,
  StepperTrigger,
} from "@/components/ui/stepper";
import { Check, Circle, Dot, RotateCcw, X } from "lucide-vue-next";

const id = useId();

const steps = [1, 2, 3, 4, 5];
const currentStep = ref(1);
const userAnswers = ref<Record<number, number>>({});
const showResults = ref(false);
const answersSubmitted = ref<Record<number, boolean>>({});

const quizList = ref([
  {
    questionNumber: 1,
    question: "What does HTML stand for?",
    options: [
      {
        id: 1,
        label: "Hyper Trainer Marking Language",
        isRight: false,
      },
      {
        id: 2,
        label: "HyperText Markup Language",
        isRight: true,
      },
      {
        id: 3,
        label: "HyperText Machine Language",
        isRight: false,
      },
      {
        id: 4,
        label: "HighText Markup Level",
        isRight: false,
      },
    ],
  },
  {
    questionNumber: 2,
    question: "What protocol is used to transfer web pages on the internet?",
    options: [
      {
        id: 1,
        label: "FTP",
        isRight: false,
      },
      {
        id: 2,
        label: "SMTP",
        isRight: false,
      },
      {
        id: 3,
        label: "HTTP",
        isRight: true,
      },
      {
        id: 4,
        label: "SSH",
        isRight: false,
      },
    ],
  },
  {
    questionNumber: 3,
    question: "What tag is used to create a hyperlink in HTML?",
    options: [
      {
        id: 1,
        label: "<link>",
        isRight: false,
      },
      {
        id: 2,
        label: "<a>",
        isRight: true,
      },
      {
        id: 3,
        label: "<href>",
        isRight: false,
      },
      {
        id: 4,
        label: "<url>",
        isRight: false,
      },
    ],
  },
  {
    questionNumber: 4,
    question: "What is the correct file extension for a JavaScript file?",
    options: [
      {
        id: 1,
        label: ".jscript",
        isRight: false,
      },
      {
        id: 2,
        label: ".js",
        isRight: true,
      },
      {
        id: 3,
        label: ".java",
        isRight: false,
      },
      {
        id: 4,
        label: ".script",
        isRight: false,
      },
    ],
  },
  {
    questionNumber: 5,
    question: "Which of the following is a valid CSS selector?",
    options: [
      {
        id: 1,
        label: "#header",
        isRight: true,
      },
      {
        id: 2,
        label: "$header",
        isRight: false,
      },
      {
        id: 3,
        label: "*header",
        isRight: false,
      },
      {
        id: 4,
        label: "header!",
        isRight: false,
      },
    ],
  },
]);

const totalScore = computed(() => {
  let score = 0;
  for (const [questionNum, answerId] of Object.entries(userAnswers.value)) {
    const question = quizList.value.find(q => q.questionNumber === parseInt(questionNum));
    const selectedOption = question?.options.find(opt => opt.id === answerId);
    if (selectedOption?.isRight) {
      score++;
    }
  }
  return score;
});

const scorePercentage = computed(() => {
  return Math.round((totalScore.value / quizList.value.length) * 100);
});

const scoreMessage = computed(() => {
  const percentage = scorePercentage.value;
  if (percentage >= 90) return "Excellent! Outstanding performance! 🌟";
  if (percentage >= 80) return "Great job! Well done! 👏";
  if (percentage >= 70) return "Good work! Keep it up! 👍";
  if (percentage >= 60) return "Not bad! You can do better! 💪";
  return "Keep studying! Practice makes perfect! 📚";
});

const selectAnswer = (answerId: number) => {
  if (!answersSubmitted.value[currentStep.value]) {
    userAnswers.value[currentStep.value] = answerId;
  }
};

const submitAnswer = () => {
  if (userAnswers.value[currentStep.value]) {
    answersSubmitted.value[currentStep.value] = true;
  }
};

const nextStep = () => {
  if (currentStep.value < steps.length) {
    currentStep.value = currentStep.value + 1;
  } else {
    showResults.value = true;
  }
};

const prevStep = () => {
  if (currentStep.value > 1) {
    currentStep.value = currentStep.value - 1;
  }
};

const restartQuiz = () => {
  currentStep.value = 1;
  userAnswers.value = {};
  showResults.value = false;
  answersSubmitted.value = {};
};

const getOptionClass = (option: any, questionNumber: number) => {
  const isSelected = userAnswers.value[questionNumber] === option.id;
  const isSubmitted = answersSubmitted.value[questionNumber];
  
  if (!isSubmitted) {
    return isSelected ? 'border-primary bg-primary/10' : 'border-input hover:bg-accent/50';
  }
  
  // After submission, show correct/incorrect colors
  if (isSelected) {
    return option.isRight 
      ? 'border-green-500 bg-green-50 text-green-700' 
      : 'border-red-500 bg-red-50 text-red-700';
  }
  
  // Show correct answer even if not selected
  if (option.isRight) {
    return 'border-green-500 bg-green-50 text-green-700';
  }
  
  return 'border-input bg-muted/30';
};

const isStepCompleted = (step: number) => {
  return answersSubmitted.value[step] === true;
};

const isStepCorrect = (step: number) => {
  if (!answersSubmitted.value[step]) return null;
  const question = quizList.value.find(q => q.questionNumber === step);
  const selectedOption = question?.options.find(opt => opt.id === userAnswers.value[step]);
  return selectedOption?.isRight || false;
};
</script>

<template>
  <main class="px-2 py-4 max-w-[800px] mx-auto">
    <h1
      class="scroll-m-20 text-4xl font-extrabold tracking-tight lg:text-5xl mb-10 text-center"
    >
      Web Basics Task
    </h1>

    <!-- Quiz Questions -->
    <template v-if="!showResults">
      <template v-for="quiz in quizList" :key="quiz.questionNumber">
        <Card v-if="quiz.questionNumber === currentStep">
          <CardHeader>
            <CardTitle>Question {{ quiz.questionNumber }}.</CardTitle>
            <CardDescription>
              {{ quiz.question }}
            </CardDescription>
          </CardHeader>
          <CardContent>
            <form @submit.prevent="submitAnswer">
              <RadioGroup 
                class="gap-2" 
                :model-value="userAnswers[currentStep]?.toString()"
                @update:model-value="selectAnswer(parseInt($event))"
                :disabled="answersSubmitted[currentStep]"
              >
                <div
                  v-for="option in quiz.options"
                  :key="option.id"
                  :class="[
                    'relative flex w-full items-center gap-2 rounded-md border px-4 py-3 shadow-xs outline-none transition-all cursor-pointer',
                    getOptionClass(option, quiz.questionNumber)
                  ]"
                >
                  <RadioGroupItem
                    :value="option.id.toString()"
                    :id="`${id}-${option.label}`"
                    :aria-describedby="`${id}-${option.label}-description`"
                    class="order-1 after:absolute after:inset-0"
                    :disabled="answersSubmitted[currentStep]"
                  />
                  <div class="grid grow gap-1">
                    <Label 
                      :htmlFor="`${id}-${option.label}`"
                      class="cursor-pointer font-medium"
                    >
                      {{ option.label }}
                      <span v-if="answersSubmitted[currentStep] && option.isRight" class="ml-2 text-green-600">✓</span>
                      <span v-if="answersSubmitted[currentStep] && !option.isRight && userAnswers[currentStep] === option.id" class="ml-2 text-red-600">✗</span>
                    </Label>
                  </div>
                </div>
              </RadioGroup>
              
              <div class="mt-6">
                <Button 
                  v-if="!answersSubmitted[currentStep]"
                  type="submit"
                  :disabled="!userAnswers[currentStep]"
                  class="w-full"
                >
                  Submit Answer
                </Button>
                <div v-else class="text-center">
                  <p class="text-sm text-muted-foreground mb-4">
                    {{ userAnswers[currentStep] && quiz.options.find(opt => opt.id === userAnswers[currentStep])?.isRight 
                        ? '🎉 Correct! Well done!' 
                        : '❌ Incorrect. The correct answer is highlighted above.' }}
                  </p>
                </div>
              </div>
            </form>
          </CardContent>
        </Card>
      </template>
    </template>

    <!-- Results Page -->
    <Card v-if="showResults" class="text-center">
      <CardHeader>
        <CardTitle class="text-3xl">Quiz Complete! 🎊</CardTitle>
        <CardDescription class="text-lg">
          Here are your results
        </CardDescription>
      </CardHeader>
      <CardContent class="space-y-6">
        <div class="text-6xl font-bold text-primary">
          {{ totalScore }}/{{ quizList.length }}
        </div>
        <div class="text-2xl font-semibold">
          {{ scorePercentage }}%
        </div>
        <p class="text-lg text-muted-foreground max-w-md mx-auto">
          {{ scoreMessage }}
        </p>
        
        <!-- Detailed Results -->
        <div class="mt-8 space-y-4 text-left max-w-2xl mx-auto">
          <h3 class="text-xl font-semibold text-center mb-4">Review Your Answers:</h3>
          <div v-for="quiz in quizList" :key="quiz.questionNumber" class="border rounded-lg p-4">
            <h4 class="font-medium mb-2">{{ quiz.questionNumber }}. {{ quiz.question }}</h4>
            <div class="grid gap-2">
              <div v-for="option in quiz.options" :key="option.id" class="flex items-center gap-2 text-sm">
                <span v-if="userAnswers[quiz.questionNumber] === option.id && option.isRight" class="text-green-600">✓</span>
                <span v-else-if="userAnswers[quiz.questionNumber] === option.id && !option.isRight" class="text-red-600">✗</span>
                <span v-else-if="option.isRight" class="text-green-600">✓</span>
                <span v-else class="w-4"></span>
                <span :class="{
                  'text-green-600 font-medium': option.isRight,
                  'text-red-600': userAnswers[quiz.questionNumber] === option.id && !option.isRight
                }">
                  {{ option.label }}
                </span>
              </div>
            </div>
          </div>
        </div>

        <Button @click="restartQuiz" class="mt-6" size="lg">
          <RotateCcw class="w-4 h-4 mr-2" />
          Restart Quiz
        </Button>
      </CardContent>
    </Card>

    <!-- Stepper and Navigation -->
    <div class="mx-auto max-w-xl space-y-8 text-center mt-5" v-if="!showResults">
      <Stepper v-model="currentStep" class="flex w-full items-start gap-2">
        <StepperItem
          v-for="step in steps"
          :key="step"
          v-slot="{ state }"
          class="relative flex w-full flex-col items-center justify-center"
          :step="step"
        >
          <StepperSeparator
            v-if="step !== steps[steps.length - 1]"
            :class="[
              'absolute left-[calc(50%+20px)] right-[calc(-50%+10px)] top-5 block h-0.5 shrink-0 rounded-full bg-muted',
              isStepCompleted(step) ? 'bg-primary' : ''
            ]"
          />

          <StepperTrigger as-child>
            <Button
              :variant="
                isStepCompleted(step) || state === 'active'
                  ? 'default'
                  : 'outline'
              "
              size="icon"
              class="z-10 rounded-full shrink-0"
              :class="[
                state === 'active' &&
                  'ring-2 ring-ring ring-offset-2 ring-offset-background',
              ]"
            >
              <Check v-if="isStepCompleted(step) && isStepCorrect(step) === true" class="size-5" />
              <X v-else-if="isStepCompleted(step) && isStepCorrect(step) === false" class="size-5" />
              <Circle v-else-if="state === 'active'" />
              <Dot v-else />
            </Button>
          </StepperTrigger>
        </StepperItem>
      </Stepper>
      
      <div class="flex justify-center space-x-4">
        <Button
          variant="outline"
          class="w-32"
          @click="prevStep"
          :disabled="currentStep === 1"
        >
          Prev step
        </Button>
        <Button
          variant="outline"
          class="w-32"
          @click="nextStep"
          :disabled="!answersSubmitted[currentStep]"
        >
          {{ currentStep === steps.length ? 'Show Results' : 'Next step' }}
        </Button>
      </div>
    </div>

    <p
      class="text-muted-foreground mt-8 text-xs text-center"
      role="region"
      aria-live="polite"
    >
      Made with ❤️ by Muslikhiddin Kuchkorov
    </p>
  </main>
</template>