<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, watchEffect } from 'vue'
import VChart from '@visactor/vchart'
import type { ISpec } from '@visactor/vchart'
import { convertVChartToSvg } from '@visactor/vchart-svg-plugin';

const container = ref<HTMLDivElement | null>(null)
let vchart: VChart | null = null

// 🎨 Палитра (из твоего варианта, без изменений)
const COLORS = {
  opened: '#2196F3',
  bg_opened: '#E3F2FD',

  clicked: '#FFC340',
  bg_clicked: '#F5A623',

  sent_notify: '#2E93E5',
  sent_notify_alarm: '#ECC424',
  bg_sent_notify: '#A7D3F2',

  no_sent_notify: '#ffae00',
  no_sent_notify_alarm: '#FF7400',
  bg_no_sent_notify: '#A7D3F2',

  submitted: '#F44336',
  bg_submitted: '#FFF3E0',

  sessions: '#D84A4A',
  bg_sessions: '#FFEBEE',

  pswd_change: '#FFEBEE',
  bg_pswd_change: '#FFEBEE',

  no_pswd_change: '#7A1E1E',
  bg_no_pswd_change: '#FFEBEE',

  white: '#FFFFFF',
  black: '#000000'
} as const

// 🎯 Функция разрешения цвета — используем строго твою логику
function resolveColor(input?: string): string | undefined {
  if (!input) return undefined;
  const s = input.trim().toLowerCase();
  if (COLORS[s as keyof typeof COLORS]) return COLORS[s as keyof typeof COLORS];
  if (/^#([0-9a-f]{3}|[0-9a-f]{6})$/i.test(s)) return s;
  if (/^rgba?\(/i.test(s)) return s;
  return undefined;
}

// 📊 Данные (точно из твоего примера)
const data = [
  {
    id: 'id0',
    values: [
      { type: 'Переходов по ссылке', value: 42, color: 'clicked' },
      { type: 'Переходов по ссылке', value: 7, color: 'clicked' },
      { type: 'Переходов по ссылке', value: 336 - 132 - 49, color: 'clicked' },
      { type: 'Переходов по ссылке', value: 132, color: 'clicked' },
    ]
  },
  {
    id: 'id1',
    values: [
      { type: 'Не уведомили ИБ', value: 204, color: 'no_sent_notify' },
      { type: 'Уведомили ИБ', value: 132, color: 'clicked' },
    ]
  },
  {
    id: 'id2',
    values: [
      { type: 'Не уведомили ИБ ', value: 42, color: 'no_sent_notify_alarm' },
      { type: 'Не уведомили ИБ ', value: 7, color: 'no_sent_notify_alarm' },
      { type: 'Не уведомили ИБ', value: 336 - 132 - 49, color: 'no_sent_notify' },
      { type: 'Уведомили ИБ', value: (132 - 7), color: 'sent_notify' },
      { type: 'Уведомили ИБ ', value: 7, color: 'sent_notify_alarm' },
    ]
  },
  {
    id: 'id3',
    values: [
      { type: 'Перехвачено данных', value: 42, color: 'submitted' },
      { type: 'Перехвачено данных', value: 7, color: 'submitted' },
      { value: 280, color: 'white' },
      { type: 'Перехвачено данных', value: 7, color: 'submitted' },
    ]
  },
  {
    id: 'id4',
    values: [
      { type: 'Перехвачено сессий', value: 42, color: 'sessions', showLabel: false },
      { type: 'Перехвачено сессий', value: 5, color: 'sessions' },
      { value: 282, color: 'white' },
      { type: 'Перехвачено сессий', value: 7, color: 'sessions' },
    ]
  },
  {
    id: 'id5',
    values: [
      { id: 1, type: 'Не сменило пароль', value: 42, color: 'no_pswd_change' },
      { value: 290, color: 'white' },
      { id: 2, type: 'Не сменило пароль', value: 4, color: 'no_pswd_change' }
    ]
  }
];

const FILLER = '__white__'

// проставляем type плейсхолдерам (там, где его нет)
for (const ds of data) {
  ds.values = ds.values.map(v => (v.type ? v : { ...v, type: FILLER }))
}

// пересобираем сопоставление type -> color, включая плейсхолдер
const typeToColor: Record<string, string> = {}
for (const ds of data) {
  for (const v of ds.values) {
    const resolved = resolveColor(v.color)
    if (resolved && v.type) typeToColor[v.type] = resolved
  }
}
// явно фиксируем белый для тех.типа
typeToColor[FILLER] = '#FFFFFF'

const domain = Object.keys(typeToColor)
const range = domain.map(d => typeToColor[d])

// 📈 Конфигурация графика
const spec: ISpec = {
  type: 'pie',
  data,
  scales: [
    {
      id: 'color',
      type: 'ordinal',
      field: 'type',
      domain,
      range
    }
  ],
  series: [
    {
      type: 'pie',
      dataIndex: 0,
      outerRadius: 0.43,
      innerRadius: 0.35,
      valueField: 'value',
      categoryField: 'type',
      padAngle: 0.08,
      label: {
        visible: false,
        formatter: '{type}',
        position: 'inside',
        rotate: false
      },
      pie: {
        style: { stroke: '#ffffff', lineWidth: 2 }
      }
    },
    {
      type: 'pie',
      dataIndex: 1,
      outerRadius: 0.5,
      innerRadius: 0.45,
      valueField: 'value',
      categoryField: 'type',
      padAngle: 0.08,
      label: {
        visible: true,
        formatter: '{_percent_}%',
        position: 'inside-inner',
        offsetRadius: 0,
        rotate: false,
        line: { visible: true },
        style: {
          stroke: 'white',
          lineWidth: 3,
          strokeOpacity: 0,
          fontSize: 14,
          fontWeight: 'bold'
        }
      },
      pie: {
        style: { stroke: '#ffffff', lineWidth: 2 }
      }
    },
    {
      type: 'pie',
      dataIndex: 2,
      outerRadius: 0.6,
      innerRadius: 0.47,
      valueField: 'value',
      categoryField: 'type',
      padAngle: 0.08,
      label: {
        visible: true,
        formatter: '{_percent_}%',
        position: 'outside',
        rotate: false,
        offsetRadius: 20,
        line: {
          visible: true,
          smooth: false,
        },
        style: {
          stroke: "white",
          lineWidth: 3,
          strokeOpacity: 1,
          fontSize: 14,
          fontWeight: 'bold'
        }
      },
      pie: {
        style: { stroke: '#ffffff', lineWidth: 2 }
      }
    },
    {
      type: 'pie',
      dataIndex: 3,
      outerRadius: 0.78,
      innerRadius: 0.62,
      valueField: 'value',
      categoryField: 'type',
      padAngle: 0.08,
      label: {
        visible: true,
        formatter: '{value} ({_percent_}%)',
        position: 'outside',
        style: {
          stroke: "white",
          lineWidth: 3,
          strokeOpacity: 1,
          fontSize: 14,
          fontWeight: 'bold'
        }
      },
      pie: {
        style: { stroke: '#ffffff', lineWidth: 2 }
      }
    },
    {
      type: 'pie',
      dataIndex: 4,
      outerRadius: 1,
      innerRadius: 0.8,
      valueField: 'value',
      categoryField: 'type',
      padAngle: 0.08,
      label: {
        visible: true,
        showEmptyCircle: true,
        formatter: '{_percent_}%',
        position: 'outside',
        style: {
          stroke: "white",
          lineWidth: 3,
          strokeOpacity: 1,
          fontSize: 14,
          fontWeight: 'bold'
        }
      },
      pie: {
        style: { stroke: '#ffffff', lineWidth: 2 }
      }
    },
    {
      type: 'pie',
      dataIndex: 5,
      outerRadius: 1.25,
      innerRadius: 1.02,
      valueField: 'value',
      categoryField: 'type',
      padAngle: 0.08,
      label: {
        visible: true,
        formatter: '{_percent_}%',
        position: 'outside',
        style: {
          stroke: "white",
          lineWidth: 3,
          strokeOpacity: 1,
          fontSize: 14,
          fontWeight: 'bold'
        }
      },
      pie: {
        style: { stroke: '#ffffff', lineWidth: 2 }
      }
    }
  ],
  indicator: {
    visible: true,
    trigger: 'hover',
    limitRatio: 0.4, // same as inner radius
    title: {
      visible: true,
      autoFit: true,
      fitStrategy: 'inscribed',
      style: {
        fontWeight: 'bolder',
        fontFamily: 'Times New Roman',
        fill: '#888',
        text: datum => {
          const d = datum ?? data[0];
          return d['formula'];
        }
      }
    },
    content: [
      {
        visible: true,
        autoFit: false,
        fitStrategy: 'inscribed',
        style: {
          fontSize: 50,
          fill: 'black',
          fontWeight: 'bolder',
          fontFamily: 'Times New Roman',
          text: "366"
        }
      },
      {
        visible: true,
        autoFit: false,
        fitStrategy: 'inscribed',
        style: {
          fill: 'black',
          fontSize: 15,
          fontFamily: 'Times New Roman',
          text: "Переходов по ссылке"
        }
      },
      {
        visible: false,
        autoFit: true,
        fitStrategy: 'inscribed',
        style: {
          fill: 'orange',
          fontFamily: 'Times New Roman',
          text: datum => {
            const d = datum ?? data[0];
            return d['value'] + '%';
          }
        }
      }
    ]
  },
  title: {
    visible: false,
    text: 'Population Distribution by Age in the United States, 2021 (in millions)',
    textStyle: {
      fontFamily: 'Times New Roman'
    }
  },
  legends: {
    visible: true,
    orient: 'left',
    items: domain
      .filter(label => label !== FILLER)
      .map(label => ({ label, shape: { fill: typeToColor[label] } }))
  },
} as unknown as ISpec
function downloadTextAsFile(text: string, filename = 'image.svg') {
  const blob = new Blob([text], { type: 'image/svg+xml;charset=utf-8' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = filename
  document.body.appendChild(a)
  a.click()
  a.remove()
  URL.revokeObjectURL(url)
}


function vchartToPngDataURL(vchart: any): string | null {
  const stage = vchart?.getStage?.();
  const canvas = stage?.getNativeHandler?.() ?? stage?.toCanvas() // в разных версиях может отличаться
  if (!canvas) return null;
  return canvas.toDataURL('image/png');
}
// 🧩 Рендер
onMounted(async () => {
  watchEffect(async () => {
    if (!container.value) return
    // уничтожаем предыдущий инстанс, если есть
    vchart?.release?.()

    // создаем новый
    vchart = new VChart(spec, {
      dom: container.value,
      animation: false
    });

    vchart.renderSync()
    function sleep(ms: number) {
      return new Promise(resolve => setTimeout(resolve, ms));
    }

    /* await sleep(1000); */ // ждём пока график полностью отрисуется
    /* const svgContent = convertVChartToSvg(vchart)
    downloadTextAsFile(svgContent, 'image.svg') */
    /*  const dataUrl = vchartToPngDataURL(vchart);
     if (dataUrl) {
       // 1) вставить в <img>
       const img = document.createElement('img');
       img.src = dataUrl;
       document.body.appendChild(img);
 
       // 2) скачать как файл
       const a = document.createElement('a');
       a.href = dataUrl;
       a.download = 'chart.png';
       a.click();
     } */

  })

  function sleep(ms: number) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
})



onBeforeUnmount(() => vchart?.release?.())
</script>

<template>
  
  <div 
    ref="container"
    style="width: 800px; height: 500px; margin: auto; display: block"
  ></div>
  
</template>
