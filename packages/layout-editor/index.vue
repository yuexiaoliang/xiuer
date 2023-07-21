<script lang="ts">
export default {
  name: 'XiuerLayoutEditor'
};
</script>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';

const pointerInfo = ref({ x: 0, y: 0 });
const targetElement = ref<HTMLElement | null>(null);

const layoutRef = ref<HTMLElement | null>(null);
const rulerDirection = ref<'horizontal' | 'vertical'>('horizontal');

interface RulerStyle {
  width: string;
  height: string;
  left: string;
  right: string;
  top: string;
  bottom: string;
}

const rulerStyle = computed<RulerStyle | null>(() => {
  return setRulerStyle();

  // 设置标尺样式
  function setRulerStyle() {
    const thick = 10;
    const { x, y } = pointerInfo.value;

    if (rulerDirection.value === 'horizontal') {
      return {
        width: `100%`,
        height: `${thick}px`,
        left: '0',
        right: '0',
        top: `${y}px`,
        bottom: 'auto'
      };
    }

    if (rulerDirection.value === 'vertical') {
      return {
        width: `${thick}px`,
        height: `100%`,
        left: `${x}px`,
        right: 'auto',
        top: '0',
        bottom: '0'
      };
    }

    return null;
  }
});

const rulerStyleWidth = computed(() => {
  return rulerStyle?.value?.width;
});

const rulerStyleHeight = computed(() => {
  return rulerStyle?.value?.height;
});

const rulerStyleLeft = computed(() => {
  return rulerStyle?.value?.left;
});

const rulerStyleRight = computed(() => {
  return rulerStyle?.value?.right;
});

const rulerStyleTop = computed(() => {
  return rulerStyle?.value?.top;
});

const rulerStyleBottom = computed(() => {
  return rulerStyle?.value?.bottom;
});

onMounted(() => {
  const layout = layoutRef.value;

  if (!layout) return;

  layout.addEventListener('mousemove', (e) => {
    const { clientX, clientY, target } = e;
    if (!isHTMLElement(target)) return;

    const { left, top } = target.getBoundingClientRect();

    pointerInfo.value = {
      x: clientX - left,
      y: clientY - top
    };

    if (targetElement.value) {
      targetElement.value.removeAttribute('id');
    }

    target.setAttribute('id', 'x__layout-editor-target');

    targetElement.value = e.target as HTMLElement;

    targetElement.value.addEventListener('mouseleave', () => {
      targetElement.value?.removeAttribute('id');
      targetElement.value = null;
    });
  });

  layout.addEventListener('mouseenter', () => {
    document.addEventListener('keydown', onShiftDown);
    document.addEventListener('keyup', onShiftUp);
  });

  layout.addEventListener('mouseleave', () => {
    document.removeEventListener('keydown', onShiftDown);
    document.removeEventListener('keyup', onShiftUp);
  });
});

// 按下 shift 键
const onShiftDown = (e: KeyboardEvent) => {
  if (e.key !== 'Shift') return;

  if (rulerDirection.value === 'vertical') return;

  rulerDirection.value = 'vertical';
};

// 松开 shift 键
const onShiftUp = (e: KeyboardEvent) => {
  if (e.key !== 'Shift') return;

  if (rulerDirection.value === 'horizontal') return;

  rulerDirection.value = 'horizontal';
};

// 是否为 HTMLElement
function isHTMLElement(target: any): target is HTMLElement {
  return target instanceof HTMLElement;
}
</script>

<template>
  <div class="x__layout-editor" ref="layoutRef">
    <div style="position: relative; height: 50%; width: 100%">
      <div style="position: relative; width: 33.3333%; height: 100%; float: left"></div>
      <div style="position: relative; width: 33.3333%; height: 100%; float: left"></div>
      <div style="position: relative; width: 33.3333%; height: 100%; float: left"></div>
    </div>
    <div style="position: relative; height: 50%; width: 100%">2</div>
  </div>
</template>

<style lang="scss" scoped>
.x__layout-editor {
  width: 100%;
  height: 100%;
  background-color: red;
}

#x__layout-editor-target {
  &::after {
    content: '';
    position: absolute;
    left: v-bind(rulerStyleLeft);
    right: v-bind(rulerStyleRight);
    top: v-bind(rulerStyleTop);
    bottom: v-bind(rulerStyleBottom);
    z-index: 99999;
    width: v-bind(rulerStyleWidth);
    height: v-bind(rulerStyleHeight);
    background-color: rgba($color: #000000, $alpha: 0.8);
  }
}
</style>
