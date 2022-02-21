<template>
  <div ref="containerRef" class="virtial-container" @scroll="onScroll">
    <div class="list-height" :style="{ height: listHeight + 'px' }"></div>
    <div ref="listRef" class="list" :style="{ transform: `translate3d(0, ${offsetTop}px, 0)` }">
      <div
        v-for="(item, index) in visibleList"
        :key="index"
        class="list-item"
        :style="{ minHeight: itemHeight + 'px', height: autoHeight ? 'auto' : itemHeight + 'px' }"
      >
        <slot :item="item" :index="startIndex + index"></slot>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick, computed, watch, toRefs } from 'vue';

const props = defineProps({
  list: {
    type: Array,
    default: () => [],
  },
  // 列表每项的高度，开启自动高度时，此为最小高度
  itemHeight: {
    type: Number,
    default: 100,
  },
  // 是否自动计算高度
  autoHeight: {
    type: Boolean,
    default: false,
  },
  // 触底事件触发时距页面底部距离，单位为px
  reachBottomDistance: {
    type: Number,
    default: 50,
  },
});

const emit = defineEmits(['reachBottom']);

const { list, itemHeight, autoHeight, reachBottomDistance } = toRefs(props);

const containerRef = ref(null);
const listRef = ref(null);

const containerHeight = ref(0);
const startIndex = ref(0);
const endIndex = ref(0);
const positions = ref([]);
const isReachBottom = ref(false);

let _throttleScroll = () => {};

onMounted(() => {
  nextTick(() => {
    containerHeight.value = containerRef.value.getBoundingClientRect().height;
    update();
  });

  _throttleScroll = throttle(update, 16);
});

const listHeight = computed(() => {
  let height = 0;
  for (let i = 0; i < list.value.length; i++) {
    height += positions.value[i] || itemHeight.value;
  }
  return height;
});

const offsetTop = computed(() => {
  let height = 0;
  for (let i = 0; i < startIndex.value; i++) {
    height += positions.value[i] || itemHeight.value;
  }
  return height;
});

const visibleList = computed(() => {
  return list.value.slice(startIndex.value, endIndex.value);
});

watch(list, (newVal, oldVal) => {
  if (newVal.length && !oldVal.length) {
    update();
  }
});

const throttle = (func, time) => {
  let date = 0;
  return (...args) => {
    const newDate = new Date();
    if (newDate - date >= time) {
      func.apply(this, args);
      date = newDate;
    }
  };
};

const getOffsetIndex = (height = 0) => {
  let count = 0;
  for (let i = 0; i < list.value.length; i++) {
    const num = positions.value[i] || itemHeight.value;
    if (count + num >= height) return i;
    count += num;
  }
  return 0;
};

const update = (scrollTop = 0) => {
  const newStartIndex = getOffsetIndex(scrollTop);
  const newEndIndex = newStartIndex + getOffsetIndex(containerHeight.value) + 1;
  const redundance = Math.ceil((newEndIndex - newStartIndex) / 2);

  startIndex.value = newStartIndex - redundance < 0 ? 0 : newStartIndex - redundance;
  endIndex.value = newEndIndex + redundance > list.value.length ? list.value.length : newEndIndex + redundance;

  updatePositions();
};

const updatePositions = () => {
  if (!autoHeight.value || !list.value.length) return;
  for (let i = 0; i <= endIndex.value - startIndex.value; i++) {
    const index = startIndex.value + i;
    if (positions.value[index]) continue;
    const node = listRef.value.children[i];
    if (node) {
      positions.value.splice(index, 0, node.getBoundingClientRect().height);
    }
  }
};

const onScroll = (e) => {
  _throttleScroll(e.target.scrollTop);
  if (listHeight.value - containerHeight.value - e.target.scrollTop <= reachBottomDistance.value) {
    if (!isReachBottom.value) {
      isReachBottom.value = true;
      emit('reachBottom');
    }
  } else {
    isReachBottom.value = false;
  }
};
</script>

<style lang="scss" scoped>
.virtial-container {
  position: relative;
  height: 100%;
  overflow-y: auto;
  -webkit-overflow-scrolling: touch;
}
.list-height {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
}
.list {
  .list-item {
    border-bottom: 1px solid #ccc;
    overflow: hidden;
    box-sizing: border-box;
  }
}
</style>
