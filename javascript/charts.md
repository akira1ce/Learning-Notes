# 3D 环形饼图

```Vue
<script setup lang="ts">
import { BasicShadowMap, SRGBColorSpace, NoToneMapping } from 'three';
import { TresCanvas } from '@tresjs/core';
import { OrbitControls } from '@tresjs/cientos';
import { Shape, ExtrudeGeometry, Vector3, Path } from 'three';
import {onMounted,reactive} from 'vue'

const gl = {
  clearColor: '#82DBC5',
  shadows: true,
  alpha: false,
  shadowMapType: BasicShadowMap,
  outputColorSpace: SRGBColorSpace,
  toneMapping: NoToneMapping,
};

function generateSlices(data) {
  const totalAngle = 2 * Math.PI; // 完整圆的角度
  let currentAngle = 0;
  const slices = [];

  for (let i = 0; i < data.length; i++) {
    const sliceAngle = (data[i].value / data.reduce((acc, curr) => acc + curr.value, 0)) * totalAngle;
    slices.push({
      color: data[i].color,
      value: data[i].value,
      startAngle: currentAngle,
      endAngle: currentAngle + sliceAngle,
    });
    currentAngle += sliceAngle;
  }

  return slices;
}

const data = reactive([
  { title: 'I',color: '#82DBC5', value: 0 },
  { title: 'II',color: '#4F4F4F', value: 2 },
  { title: 'II',color: '#FBB03B', value: 1.5 },
  { title: 'IV',color: '#FF6347', value: 0.5 },
]);

const slices = generateSlices(data);

function createSlice({ startAngle, endAngle, value }, outerRadius, innerRadius) {
  const shape = new Shape();
  shape.moveTo(outerRadius * Math.cos(startAngle), outerRadius * Math.sin(startAngle));
  shape.absarc(0, 0, outerRadius, startAngle, endAngle, false);
  shape.lineTo(innerRadius * Math.cos(endAngle), innerRadius * Math.sin(endAngle));
  shape.absarc(0, 0, innerRadius, endAngle, startAngle, true);
  shape.closePath();
  return [shape, {
    depth: value,
    bevelEnabled: false,
  }];
}

onMounted(()=>{
data[0].value = 1
console.log(data)
})
</script>

<template>
  <TresCanvas v-bind="gl">
    <TresPerspectiveCamera :position="[9, 9, 9]" />
    <OrbitControls />
    <template v-for="(slice, index) in slices" :key="index">
      <TresMesh :position="[0, 0, 0]" cast-shadow>
        <TresExtrudeGeometry :args="createSlice(slice, 2, 1)" />
        <TresMeshToonMaterial :color="slice.color" :transparent="true" :opacity="0.8" />
      </TresMesh>
    </template>
    <TresDirectionalLight :position="[0, 2, 4]" :intensity="1.2" cast-shadow />
  </TresCanvas>
</template>

<style>
html,
body {
  margin: 0;
  padding: 0;
  height: 100%;
  width: 100%;
}
#app {
  height: 100%;
  width: 100%;
  background-color: #000;
}
</style>
```

![alt text](assets/image.png)
