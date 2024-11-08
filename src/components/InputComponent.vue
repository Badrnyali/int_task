<template>
  <div class="q-gutter-sm">
    <q-radio v-model="regexEngines" val="pcre" label="PCRE" />
    <q-radio v-model="regexEngines" val="re2js" label="RE2JS" />
  </div>
  <q-input
    ref="inputRef"
    square
    flat
    stack-label
    filled
    required
    v-model="cpPattern"
    :label="label"
    placeholder="(Hello)?\s?(World)?"
    type="text"
    :error="!isValid"
    hide-bottom-space
  >
    <template #prepend>
      <IconRegex stroke={2} />
    </template>
    <template #append>
      <div class="cursor-pointer">
        <span
          class="text-body1 text-mute"
          style="font-family: SourceCodePro, monospace"
          >/{{ flags?.replace(/g/, '') }}
        </span>
        <q-popup-proxy transition-show="scale" transition-hide="scale">
          <div
            class="column q-px-md q-py-md q-gutter-sm"
            style="line-height: 1em"
          >
            <div>Regex Flags</div>
            <q-checkbox
              class="text-caption"
              v-model="flagsChecked.multiline"
              dense
              @click="updateFlags"
            >
              Multiline
            </q-checkbox>
            <q-checkbox
              class="text-caption"
              v-model="flagsChecked.caseInsensitive"
              dense
              @click="updateFlags"
            >
              Case Insensitive
            </q-checkbox>
          </div>
        </q-popup-proxy>
      </div>
    </template>
    <template #error>
      <p>RegExp Syntax Error</p>
    </template>
  </q-input>
</template>

<script setup lang="ts">
import { IconRegex } from '@tabler/icons-vue';
import { ref, reactive, computed, useTemplateRef, watch } from 'vue';
import { RE2JS } from 're2js';

const emits = defineEmits<{
  (e: 'updateRaw', newRaw: string): void;
  (e: 'updateFlags', newFlags: string): void;
}>();

const inputRef = useTemplateRef('inputRef');
const isValid = ref(true);

const props = defineProps<{
  label: string;
  flags: string;
}>();

const cpPattern = defineModel<string>();

//RegexEngine can either
const regexEngines = ref<'pcre' | 're2js'>('pcre');

const flagsChecked = reactive({
  multiline: Boolean(props.flags?.indexOf('m') > -1),
  caseInsensitive: Boolean(props.flags?.indexOf('i') > -1),
});


const thisFlags = computed(() => {
  let flags = '';

  if (flagsChecked.multiline) {
    flags += 'm';
  }

  if (flagsChecked.caseInsensitive) {
    flags += 'i';
  }
  return flags;
});

//Validate Regex depending on regex engine
const validateRegex = () => {
  if (regexEngines.value === 're2js') {
    try {
      RE2JS.compile(String(cpPattern.value));
      return true;
    } catch (err) {
      console.log('error re2');
      return false;
    }
  }
  try {
    new RegExp(String(cpPattern.value));
    return true;
  } catch (err) {
    console.log('error pcre');
    return false;
  }
};

//handling and emiting regex syntax
const handleValidation = () => {
  isValid.value = true;
  const isValidRegex = validateRegex();
  if (!isValidRegex) {
    emits('updateRaw', '');
    isValid.value = false;
    return;
  }
  const validRegex = new RegExp(String(cpPattern.value), thisFlags.value);
  emits('updateRaw', validRegex.toString());
};

const updateFlags = () => {
  emits('updateFlags', thisFlags.value);
  handleValidation();
};

watch(regexEngines, handleValidation);
watch(cpPattern, handleValidation);
</script>

<style scoped lang="scss">
:deep(input) {
  font-family: SourceCodePro, monospace;
  letter-spacing: -0.75px;
}
</style>
