<template>
  <q-page class="col items-center justify-evenly">
    <InputComponent
      :label="label"
      :flags="flags"
      v-model="pattern"
      @updateFlags="updateFlags"
      @updateRaw="updateSyntax"
    />
    <p v-if="syntax.length">Regex Syntax: {{ syntax }} {{ syntax.length }}</p>
  </q-page>
</template>

<script setup lang="ts">
import InputComponent from 'src/components/InputComponent.vue';
import { ref } from 'vue';

defineOptions({
  name: 'IndexPage',
});

const label = ref<string>('Regular Expression');
const pattern = ref<string>('(Hello)?\s?(World)?');
const flags = ref<string>('i');
const syntax = ref<string>(String(new RegExp(pattern.value, flags.value)));

function updateFlags(newFlags: string) {
  flags.value = newFlags;
}

function updateSyntax(newSyntax: string) {
  syntax.value = newSyntax;
}
</script>
