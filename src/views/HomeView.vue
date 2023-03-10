<script setup lang="ts">
import { reactive, computed } from 'vue'
import { openaiApi } from '@/lib/openaiApi';
import Formats from '../lib/formats'
import ImageDisplay from '@/components/ImageDisplay.vue';
import LoadingSpinner from '@/components/LoadingSpinner.vue';

const MAX_PROMPT_LENGTH = 1000

const state = reactive({
  formats: Object.values(Formats),
  selectedOption: Formats.SMALL,
  prompt: '',
  imageUrl: '',
  isLoading: false,
  rules: {
    prompt: [(v: string | any[]) => v.length <= MAX_PROMPT_LENGTH || `Max ${MAX_PROMPT_LENGTH} characters`]
  }
})

const hasImage = computed(() => {
  return state.imageUrl.length > 0
})

const loadImage = async () => {
  try {
    state.isLoading = true
    state.imageUrl = ''
    const response = await openaiApi.createImage({
      prompt: state.prompt,
      n: 1,
      size: state.selectedOption,
      response_format: 'url'
    })

    if (response && response.status === 200) {
      state.isLoading = false
      state.imageUrl = response.data.data[0].url || ''
    }
    
  } catch (error) {
    console.log('Something went horribly wrong:', error)
  }

}
</script>

<template>
  <main>
    <section class="promt">
      <VTextarea
        clearable 
        label='Please type your image description'
        :rules="state.rules.prompt"
        v-model="state.prompt"
      />
    </section>
    
    <section class="format">
      <VSelect
        label="Choose a size"
        :items="state.formats"
        v-model="state.selectedOption"
      />
    </section>

    <VBtn 
      variant="flat" 
      :disabled="state.isLoading || state.prompt.length < 1"
      @click="loadImage"
    >
      Generate Image
    </VBtn>

    <section class="image-display">
      <LoadingSpinner v-if="state.isLoading" />
      <ImageDisplay v-if="hasImage" :imageUrl="state.imageUrl" />
    </section>
  </main>
</template>

<style scoped>
.image-display {
  display: flex;
  margin-top: 10vh;
}

.image-display > .spinner-wrapper {
  margin: 0 auto;
}
</style>
