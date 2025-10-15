<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue'
import VChart from '@visactor/vchart'
import type { ISpec } from '@visactor/vchart'

const container = ref<HTMLDivElement | null>(null)
let vchart: VChart | null = null

function buildSpec(): ISpec {
  return {
    type: 'common',
    data: [
      {
        id: 'id0',
        values: [
          { type: '0~29', value: 126.04 },
          { type: '30~59', value: 128.77 },
          { type: '60 and over', value: 77.09 }
        ]
      },
      {
        id: 'id1',
        values: [
          { type: '0~9', value: 39.12 },
          { type: '10~19', value: 43.01 },
          { type: '20~29', value: 43.91 },
          { type: '30~39', value: 45.4 },
          { type: '40~49', value: 40.89 },
          { type: '50~59', value: 42.48 },
          { type: '60~69', value: 39.63 },
          { type: '70~79', value: 25.17 },
          { type: '80 and over', value: 12.29 }
        ]
      }
    ],
    series: [
      {
        type: 'pie',
        dataIndex: 0,
        outerRadius: 0.65,
        innerRadius: 0,
        valueField: 'value',
        categoryField: 'type',
        label: {
          position: 'inside',
          visible: true,
          style: { fill: 'white' }
        },
        pie: { style: { stroke: '#fff', lineWidth: 2 } }
      },
      {
        type: 'pie',
        dataIndex: 1,
        outerRadius: 0.8,
        innerRadius: 0.67,
        valueField: 'value',
        categoryField: 'type',
        label: { visible: true },
        pie: { style: { stroke: '#fff', lineWidth: 2 } }
      }
    ],
    color: [
      '#98abc5',
      '#8a89a6',
      '#7b6888',
      '#6b486b',
      '#a05d56',
      '#d0743c',
      '#ff8c00'
    ],
    title: {
      visible: true,
      text: 'Population Distribution by Age in the United States, 2021 (in millions)',
      textStyle: { fontFamily: 'Times New Roman' }
    },
    legends: {
      visible: true,
      orient: 'left'
    } as const // ← ключевая строка: фиксирует литерал 'left' как допустимый тип
  }
}

onMounted(() => {
  if (!container.value) return
  vchart = new VChart(buildSpec(), { dom: container.value })
  vchart.renderSync()
})

onBeforeUnmount(() => {
  vchart?.release?.()
})
</script>

<template>
  <div
    ref="container"
    style="width: 800px; height: 500px; margin: auto; display: block"
  ></div>
</template>
