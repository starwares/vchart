<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, watchEffect } from 'vue'
import VChart from '@visactor/vchart'
import type { ISpec } from '@visactor/vchart'

const container = ref<HTMLDivElement | null>(null)
let vchart: VChart | null = null

// 🎨 Палитра
const COLORS = {
  opened: '#2196F3',
  clicked: '#FFC340',
  sent_notify: '#2E93E5',
  no_sent_notify: '#ffae00',
  no_sent_notify_alarm: '#FF7400',
  submitted: '#F44336',
  sessions: '#D84A4A',
  no_pswd_change: '#7A1E1E',
  white: '#FFFFFF'
} as const

type ColorKey = keyof typeof COLORS

function resolveColor(input?: string): string | undefined {
  if (!input) return undefined
  const s = input.trim().toLowerCase() as ColorKey
  if (COLORS[s]) return COLORS[s]
  if (/^#([0-9a-f]{3}|[0-9a-f]{6})$/i.test(s)) return s
  if (/^rgba?\(/i.test(s)) return s
  return undefined
}

// 📊 Данные
const data = [
  {
    id: 'id0',
    values: [
      { type: 'Переходов по ссылке', value: 42, color: 'clicked' },
      { type: 'Переходов по ссылке', value: 7, color: 'clicked' },
      { type: 'Переходов по ссылке', value: 336 - 132 - 49, color: 'clicked' },
      { type: 'Переходов по ссылке', value: 132, color: 'clicked' }
    ]
  },
  {
    id: 'id1',
    values: [
      { type: 'Не уведомили ИБ ', value: 42, color: 'no_sent_notify_alarm' },
      { type: 'Не уведомили ИБ ', value: 7, color: 'no_sent_notify_alarm' },
      { type: 'Не уведомили ИБ', value: 336 - 132 - 49, color: 'no_sent_notify' },
      { type: 'Уведомили ИБ', value: 132, color: 'sent_notify' }
    ]
  },
  {
    id: 'id2',
    values: [
      { type: 'Перехвачено данных', value: 42, color: 'submitted' },
      { type: 'Перехвачено данных', value: 7, color: 'submitted' },
      { value: 280, color: 'white' },
      { type: 'Перехвачено данных', value: 7, color: 'submitted' }
    ]
  },
  {
    id: 'id3',
    values: [
      { type: 'Перехвачено сессий', value: 42, color: 'sessions' },
      { type: 'Перехвачено сессий', value: 5, color: 'sessions' },
      { value: 282, color: 'white' },
      { type: 'Перехвачено сессий', value: 7, color: 'sessions' }
    ]
  },
  {
    id: 'id4',
    values: [
      { type: 'Не сменило пароль', value: 42, color: 'no_pswd_change' },
      { value: 290, color: 'white' },
      { type: 'Не сменило пароль', value: 4, color: 'no_pswd_change' }
    ]
  }
]

// 🧩 служебные плейсхолдеры
const FILLER = '__filler__'
for (const ds of data.slice(2)) {
  ds.values = ds.values.map(v => (v.type ? v : { ...v, type: FILLER }))
}

// добавляем уникальные идентификаторы сегментам
data.forEach(ds => {
  ds.values.forEach((v, idx) => {
    v.uid = `${ds.id}_${idx}`
  })
})

// 🧠 соответствие цветов
const typeToColor: Record<string, string> = {}
for (const ds of data) {
  for (const v of ds.values) {
    const resolved = resolveColor(v.color)
    if (resolved && v.type) typeToColor[v.type] = resolved
  }
}
typeToColor[FILLER] = '#FFFFFF'

const domain = Object.keys(typeToColor)
const range = domain.map(d => typeToColor[d])

function shouldShowLabel(d: any) {
  return false
  const t = d?.type ?? d?.datum?.type
  const v = d?.value ?? d?.datum?.value
  // не показываем подписи для плейсхолдеров и белых сегментов
  if (!t || t === '__filler__' || t === 'white') return false
  // например: скрываем метки для "Перехвачено данных"
  if (v === 5 || v === 7) return true
  else
    return false
}

// 📈 Конфигурация графика
const spec: ISpec = {
  type: 'common',
  data,
  scales: [
    { id: 'color', type: 'ordinal', field: 'type', domain, range }
  ],
  series: [
    {
      type: 'pie',
      dataIndex: 0,
      outerRadius: 0.45,
      innerRadius: 0.35,
      valueField: 'value',
      categoryField: 'type',
      encode: { angle: 'value', id: 'uid' },
      padAngle: 0.45,
      pie: {
        style: (d: any) =>
          d.type === FILLER
            ? { fillOpacity: 0, strokeOpacity: 0 }
            : {
              fill: { field: 'type', scale: 'color' },
              stroke: '#fff',
              lineWidth: 2,
              cornerRadius: 3
            }
      },
      label: { visible: false }
    },
    {
      type: 'pie',
      dataIndex: 1,
      outerRadius: 0.60,
      innerRadius: 0.47,
      valueField: 'value',
      categoryField: 'type',
      encode: { angle: 'value', id: 'uid' },
      padAngle: 0.05,
      pie: {
        style: {
          fill: { field: 'type', scale: 'color' },
          stroke: '#fff',
          lineWidth: 2,
          cornerRadius: 3
        }
      },
      label: {

        visible: true,
        formatter: '{_percent_}%',
        style: { fill: '#000', fontWeight: 'bold', fontSize: 10 }
      }
    },
    {
      type: 'pie',
      dataIndex: 2,
      outerRadius: 0.78,
      innerRadius: 0.62,
      valueField: 'value',
      categoryField: 'type',
      encode: { angle: 'value', id: 'uid' },
      padAngle: 0.05,
      pie: {
        style: {
          fill: { field: 'type', scale: 'color' },
          stroke: '#fff',
          lineWidth: 2,
          cornerRadius: 3
        }
      },

      label: {


        line: {
          visible: false, // для маленьких — линия
        },
        visible: true,            // включаем метки в серии
        // formatter НЕ указываем — чтобы ничего не перетирало formatMethod

        formatMethod: (text: string, data: any) => {
          const d = data?.datum ?? data
          const t = d?.type
          const v = d?.value

          // прячем filler/white
          if (!t || t === '__filler__' || t === 'white') return undefined

          // показываем ТОЛЬКО для нужных значений (пример: 5 и 7)
          if (v === 5 || v === 7) {
            // _percent_ в v2 обычно уже в процентах (83.93), но подстрахуемся
            const raw = typeof data?._percent_ === 'number' ? data._percent_ : 0
            const perc = raw > 1.0001 ? raw : raw * 100
            return `${v}(${perc.toFixed(2)})%`
          }

          return undefined
        },
        style: { fill: '#000', fontWeight: 'bold', fontSize: 10 }
      }
    },
    {
      type: 'pie',
      dataIndex: 3,
      outerRadius: 1.0,
      innerRadius: 0.80,
      valueField: 'value',
      categoryField: 'type',
      encode: { angle: 'value', id: 'uid' },
      padAngle: 0.05,
      pie: {
        style: {
          fill: { field: 'type', scale: 'color' },
          stroke: '#fff',
          lineWidth: 2,
          cornerRadius: 3
        }
      },
      label: {
        visible: true,            // включаем метки в серии
        // formatter НЕ указываем — чтобы ничего не перетирало formatMethod
        formatMethod: (text: string, data: any) => {
          const d = data?.datum ?? data
          const t = d?.type
          const v = d?.value

          // прячем filler/white
          if (!t || t === '__filler__' || t === 'white') return ''

          // показываем ТОЛЬКО для нужных значений (пример: 5 и 7)
          if (v === 5 || v === 7) {
            // _percent_ в v2 обычно уже в процентах (83.93), но подстрахуемся
            const raw = typeof data?._percent_ === 'number' ? data._percent_ : 0
            const perc = raw > 1.0001 ? raw : raw * 100
            return `${perc.toFixed(2)}%`
          }

          // для остальных — пусто (ничего не рисуем)
          return ''
        },
        style: { fill: '#000', fontWeight: 'bold', fontSize: 10 }
      }
    },
    {
      type: 'pie',
      dataIndex: 4,
      outerRadius: 1.25,
      innerRadius: 1.02,
      valueField: 'value',
      categoryField: 'type',
      encode: { angle: 'value', id: 'uid' },
      padAngle: 0.05,
      pie: {
        style: {
          fill: { field: 'type', scale: 'color' },
          stroke: '#fff',
          lineWidth: 2,
          cornerRadius: 3
        }
      },
      label: {
        visible: true,            // включаем метки в серии
        // formatter НЕ указываем — чтобы ничего не перетирало formatMethod
        formatMethod: (text: string, data: any) => {
          const d = data?.datum ?? data
          const t = d?.type
          const v = d?.value

          // прячем filler/white
          if (!t || t === '__filler__' || t === 'white') return ''

          // показываем ТОЛЬКО для нужных значений (пример: 5 и 7)
          if (v === 4 || v === 7) {
            // _percent_ в v2 обычно уже в процентах (83.93), но подстрахуемся
            const raw = typeof data?._percent_ === 'number' ? data._percent_ : 0
            const perc = raw > 1.0001 ? raw : raw * 100
            return `${perc.toFixed(2)}%`
          }

          // для остальных — пусто (ничего не рисуем)
          return ''
        },
        style: { fill: '#000', fontWeight: 'bold', fontSize: 10 }
      }
    }
  ],
  legends: {
    visible: true,
    orient: 'left',
    items: domain
      .filter(label => label !== FILLER)
      .map(label => ({ label, shape: { fill: typeToColor[label] } }))
  },
  indicator: {
    visible: true,
    trigger: 'hover',
    title: { visible: false },
    content: [
      {
        visible: true,
        autoFit: false,
        style: {
          fill: 'black',
          fontWeight: 'bolder',
          fontFamily: 'Times New Roman',
          fontSize: 24,
          text: '366'
        }
      },
      {
        visible: true,
        autoFit: false,
        style: {
          fill: 'black',
          fontFamily: 'Times New Roman',
          fontSize: 18,
          text: 'Переходов по ссылке'
        }
      }
    ]
  }
}

// 🧩 Рендер
onMounted(() => {
  watchEffect(() => {
    if (!container.value) return
    vchart?.release?.()
    vchart = new VChart(spec, { dom: container.value })
    vchart.renderSync()
  })
})

onBeforeUnmount(() => vchart?.release?.())
</script>

<template>
  <div ref="container" style="width:800px;height:500px;margin:auto;display:block"></div>
</template>
