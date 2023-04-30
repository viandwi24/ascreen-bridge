<script lang="ts" setup>
type Args = {
  w: number
  h: number
  x: number
  y: number
  originalW: number
  originalH: number
}
const props = defineProps({
  args: {
    type: Object as () => Args,
    required: true,
  }
})

// const emit = defineEmits<{
//   (e: 'update-args', args: Args): void
// }>()

// const test = () => {
//   emit('update-args', {
//     ...props.args,
//     w: props.args.w + 100,
//   })
// }
</script>

<template>
  <div class="overflow-hidden rounded shadow">
    <div class="font-bold px-4 py-2 text-white bg-slate-900">Crop</div>
    <div class="px-4 py-2 flex flex-col bg-slate-700">
      <div class="flex">
        <div class="w-1/3">Width</div>
        <div class="flex flex-col">
          <div class="flex space-x-2">
            <div>0</div>
            <input type="range" min="0" :max="props.args.originalW" step="1" v-model="props.args.w" />
            <div>{{ props.args.originalW }}</div>
          </div>
          <div class="text-center text-sm">{{ props.args.w }}</div>
        </div>
      </div>
      <div class="flex">
        <div class="w-1/3">Height</div>
        <div class="flex flex-col">
          <div class="flex space-x-2">
            <div>0</div>
            <input type="range" min="0" :max="props.args.originalH" step="1" v-model="props.args.h" />
            <div>{{ props.args.originalH }}</div>
          </div>
          <div class="text-center text-sm">{{ props.args.h }}</div>
        </div>
      </div>
      <div class="flex">
        <div class="w-1/3">Position X</div>
        <div class="flex flex-col">
          <div class="flex space-x-2">
            <div>{{ -props.args.originalW }}</div>
            <input type="range" :min="-props.args.originalW" :max="props.args.originalW" step="1" v-model="props.args.x" />
            <div>{{ props.args.originalW }}</div>
          </div>
          <div class="text-center text-sm">{{ props.args.x }}</div>
        </div>
      </div>
      <div class="flex">
        <div class="w-1/3">Position Y</div>
        <div class="flex flex-col">
          <div class="flex space-x-2">
            <div>{{ -props.args.originalH }}</div>
            <input type="range" :min="-props.args.originalH" :max="props.args.originalH" step="1" v-model="props.args.y" />
            <div>{{ props.args.originalH }}</div>
          </div>
          <div class="text-center text-sm">{{ props.args.y }}</div>
        </div>
      </div>
    </div>
    <!-- <button @click="test">test</button> -->
  </div>
</template>