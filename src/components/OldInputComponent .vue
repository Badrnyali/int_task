<template>
  <q-input
    square
    flat
    stack-label
    filled
    required
    :model-value="cpPattern"
    @update:model-value="updateCpPattern"
    :label="label"
    placeholder="(Hello)?\s?(World)?"
    type="text"
    :error="!isValid"
    hide-bottom-space
  >
    <template #prepend>
      <!-- <Icon name="outlinedRegularExpression" size="sm" /> -->
      <p>Icon</p>
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
              v-model="multiline"
              dense
              @click="updateFlags"
            >
              Multiline
            </q-checkbox>
            <q-checkbox
              class="text-caption"
              v-model="caseInsensitive"
              dense
              @click="updateFlags"
            >
              Case Insensitive
            </q-checkbox>
          </div>
        </q-popup-proxy>
      </div>
    </template>
  </q-input>
</template>

<script setup>
import { ref, toRaw, watch } from 'vue';

// import Icon from '@/components/Icon.vue'

const props = defineProps({
  pattern: {
    type: String,
    required: false,
  },
  label: {
    type: String,
    required: false,
  },
  flags: {
    type: String,
    required: false,
  },
});

const emit = defineEmits(['update', 'updateFlags', 'updateRaw']);

const multiline = ref(props.flags?.indexOf('m') > -1);
const caseInsensitive = ref(props.flags?.indexOf('i') > -1);

const thisFlags = ref(
  `${multiline.value ? 'm' : ''}${caseInsensitive.value ? 'i' : ''}`
);

const cpPattern = ref(structuredClone(toRaw(props.pattern)));
const isValid = ref(true);

// function setValid(value) {
//   isValid.value = value;
// }

// function getValid() {
//   return isValid.value;
// }

function updateFlags() {
  let flags = '';

  if (multiline.value) {
    flags += 'm';
  }

  if (caseInsensitive.value) {
    flags += 'i';
  }

  thisFlags.value = flags;
  emit('updateFlags', flags);

  try {
    emit(
      'updateRaw',
      !cpPattern.value
        ? ''
        : new RegExp(cpPattern.value, thisFlags.value).toString()
    );
  } catch (err) {
    console.error(err);
  }
}

function updateCpPattern(value) {
  cpPattern.value = value;
  console.log('value', value);
  try {
    new RegExp(value);
  } catch (err) {
    // console.error(err)
  }

  emit('update', value);

  try {
    emit(
      'updateRaw',
      !cpPattern.value
        ? ''
        : new RegExp(cpPattern.value, thisFlags.value).toString()
    );
  } catch (err) {
    console.error(err);
  }
}

watch(cpPattern, () => {
  updateCpPattern(cpPattern.value);
});
</script>

<style scoped lang="scss">
:deep(input) {
  font-family: SourceCodePro, monospace;
  letter-spacing: -0.75px;
}
</style>
