<template>
  <div class="example">
    <div class="example__inputs">
      <input 
        v-if="state.value.not_exist.first.visible" 
        type="text" 
        placeholder="first" 
        :disabled="false"
      >
      <input 
        v-if="state.value.not_exist.second.visible" 
        type="text" 
        placeholder="second" 
        :disabled="!!state.value.not_exist.second.visible.disabled"
      >
      <input 
        v-if="state.value.not_exist.third.visible" 
        type="text" 
        placeholder="third"
        :disabled="state.value.not_exist.third.visible === 'disabled'"
      >
    </div>
    <div class="example__state">
      {{ state.value }}
    </div>
    <div class="example__buttons">
      <button @click="send('FIRST_STATE')">FIRST_STATE</button>
      <button @click="send('SECOND_STATE')">SECOND_STATE</button>
      <button @click="send('DEFAULT')">DEFAULT STATE</button>
    </div>
  </div>
</template>

<script>
import { useMachine } from 'xstate-vue2';
import { createMachine } from 'xstate';

function createInputState(id) {
  return {
    [id]: {
      id,
      initial: "visible",
      states: {
        // Состояния инпута, когда он видимый
        visible: {
          initial: 'default',
          states: {
            default: { type: 'final' },
            disabled: { type: 'final' },
            required: { type: 'final' },
          }
        },
        // Инпут скрыт
        hidden: {},
      },
      on: {
        [`${id}.DISABLED`]: { target: ".visible.disabled" },
        [`${id}.REQUIRED`]: { target: ".visible.required" },
        [`${id}.DEFAULT`]: { target: ".visible.default" },
        [`${id}.HIDDEN`]: { target: ".hidden" }
      },
    }
  };
}

const toggleMachine = createMachine({
  id: "testMachine",
  initial: "not_exist",
  states: {
    not_exist: {
      type: "parallel",
      states: {
        ...createInputState("first"),
        ...createInputState("second"),
        ...createInputState("third"),
        home_in_schedule: {
          type: "atomic",
          on: {
            DEFAULT: { target: [
              "first.visible.default",
              "second.visible.default",
              "third.visible.default"
            ]},
            FIRST_STATE: { target: [
              "first.hidden",
              "second.visible.disabled",
              "third.hidden"
            ]},
            SECOND_STATE: { target: [
              "first.visible.disabled",
              "second.hidden",
              "third.visible.disabled"
            ]},
          },
        },
      },
    },
  },
})

export default {
  setup() {
    const { state, send } = useMachine(toggleMachine);
    return {
      state,
      send
    };
  }
};
</script>

<style scoped>
.example {
  display: flex;
  flex-direction: column;
}
</style>