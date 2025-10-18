<template>
  <div class="page">
    <NuxtRouteAnnouncer />
    <div class="grid">
      <!-- 5 графиков: 3 в первой строке, 2 во второй -->
      <div v-for="i in 5" :key="i" class="cell">
        <VChartRings />
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Вся страница — на высоту окна */
.page {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

/* Грид на всю высоту окна */
.grid {
  flex: 1 1 auto;
  height: 100vh;
  /* строго в высоту экрана */
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  /* 3 колонки */
  grid-template-rows: 1fr 1fr;
  /* 2 строки одинаковой высоты */
  gap: 12px;
  /* отступы между ячейками */
  padding: 12px;
  /* внутренние отступы по краям */
  box-sizing: border-box;
  overflow: hidden;
  /* чтобы ничего не вылазило */
}

/* Карточка-ячейка под график */
.cell {
  background: #f7f7f7;
  border-radius: 12px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, .06);
  overflow: hidden;
  /* обрежем подписи, если что-то лезет за края */
  display: flex;
  align-items: stretch;
  justify-content: stretch;
}

/* Очень важно: пусть VChartRings растягивается на всю ячейку */
.cell>* {
  width: 100%;
  height: 100%;
}
</style>
