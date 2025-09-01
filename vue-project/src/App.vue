<script setup>
import { ref, computed, onMounted } from 'vue'

// Реактивные переменные
const sessions = ref([])
const searchQuery = ref('')
const currentPage = ref(1)
const itemsPerPage = ref(11)
const sortDirection = ref('asc')
const sortBy = ref('start')


const loadData = async () => {
  try {
    const response = await fetch('/data1.json')
    const data = await response.json()

    const rawSessions = data.sessions || []

    const uniqueSessions = rawSessions.filter(
        (s, i, arr) => arr.findIndex(x => x.id === s.id) === i
    )


    sessions.value = uniqueSessions.map((s, i) => ({
      ...s,
      id: s.id ?? `session-${i}`
    }))
  } catch (error) {
    console.error('Ошибка загрузки данных:', error)
    sessions.value = []
  }
}

const filteredAndSortedSessions = computed(() => {
  const query = searchQuery.value?.trim().toLowerCase() || ''

  return sessions.value
      .filter(session => {
        if (!query) return true
        const moduleName = (session.module || '').trim().toLowerCase()
        return moduleName.includes(query)
      })
      .sort((a, b) => {
        const dateA = new Date(a.start)
        const dateB = new Date(b.start)
        return sortDirection.value === 'asc' ? dateA - dateB : dateB - dateA
      })
})







const totalPages = computed(() => Math.ceil(filteredAndSortedSessions.value.length / itemsPerPage.value))

const paginatedSessions = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage.value
  const end = start + itemsPerPage.value
  return filteredAndSortedSessions.value.slice(start, end)
})

const visiblePages = computed(() => {
  const pages = []
  const maxVisible = 5
  
  if (totalPages.value <= maxVisible) {
    for (let i = 1; i <= totalPages.value; i++) {
      pages.push(i)
    }
  } else {
    if (currentPage.value <= 3) {
      for (let i = 1; i <= 5; i++) {
        pages.push(i)
      }
    } else if (currentPage.value >= totalPages.value - 2) {
      for (let i = totalPages.value - 4; i <= totalPages.value; i++) {
        pages.push(i)
      }
    } else {
      for (let i = currentPage.value - 2; i <= currentPage.value + 2; i++) {
        pages.push(i)
      }
    }
  }
  
  return pages
})


const goToPage = (page) => {
  if (page >= 1 && page <= totalPages.value) {
    currentPage.value = page
  }
}

const handleSearch = () => {
  currentPage.value = 1
}

const toggleSort = () => {
  if (sortDirection.value === 'asc') {
    sortDirection.value = 'desc'
  } else {
    sortDirection.value = 'asc'
  }
  currentPage.value = 1
}

const formatDateTime = (dateString) => {
  const date = new Date(dateString)
  return date.toLocaleDateString('ru-RU') + ' ' + date.toLocaleTimeString('ru-RU', { 
    hour: '2-digit', 
    minute: '2-digit' 
  })
}

const getStatusText = (statusName) => {
  const statusMap = {
    'planned': 'Запланировано',
    'in_progress': 'Идет',
    'completed': 'Завершено',
    'canceled': 'Отменено'
  }
  return statusMap[statusName] || statusName
}

const getStatusClass = (statusName) => {
  const classMap = {
    'planned': 'status planned',
    'in_progress': 'status progress',
    'completed': 'status done',
    'canceled': 'status canceled'
  }
  return classMap[statusName] || 'status'
}

const getTypeText = (typeName) => {
  const typeMap = {
    'accreditation': 'Аккредитация',
    'lesson': 'Урок',
    'examination': 'Экзамен'
  }
  return typeMap[typeName] || typeName
}

const getRoomNames = (rooms) => {
  if (!rooms || rooms.length === 0) return '-'
  return rooms.map(room => room.name).join(', ')
}

const getGroupNames = (groups) => {
  if (!groups || groups.length === 0) return '-'
  return groups.map(group => group.name).join(', ')
}


onMounted(() => {
  loadData()
})
</script>

<template>
  <div id="app">
    <menu>
      <div class="logo">
        <img class="logoImg" src="/icons/logo.svg" alt="logo"/>
        <div class="logoText">Сим Центр</div>
      </div>
      <div class="tabs">
        <div class="tab">
          <img class="tabIcon" src="/icons/calendar.svg" alt="menuIcon"/>
          <div class="tabText">Расписание</div>
        </div>
        <div class="tab">
          <img class="tabIcon" src="/icons/GraduationCap.svg" alt="menuIcon"/>
          <div class="tabText">Учебные сессии</div>
        </div>
        <div class="tab">
          <img class="tabIcon" src="/icons/Student.svg" alt="menuIcon"/>
          <div class="tabText">Список комнат</div>
        </div>
        <div class="tab">
          <img class="tabIcon" src="/icons/UserList.svg" alt="menuIcon"/>
          <div class="tabText">Пользователи</div>
        </div>
        <div class="tab">
          <img class="tabIcon" src="/icons/camera.svg" alt="menuIcon"/>
          <div class="tabText">Учебные группы</div>
        </div>
        <div class="tab">
          <img class="tabIcon" src="/icons/settings.svg" alt="menuIcon"/>
          <div class="tabText">Список устройств</div>
        </div>
        <div class="tab">
          <img class="tabIcon" src="/icons/download.svg" alt="menuIcon"/>
          <div class="tabText">Настройки системы</div>
        </div>
        <div class="tab">
          <img class="tabIcon" src="/icons/calendar.svg" alt="menuIcon"/>
          <div class="tabText">Архив</div>
        </div>
      </div>

      <div class="user">
        <div class="userInfo">
          <div class="userName">Барнаби Мармадюк</div>
          <div class="userStatus">Преподаватель</div>
        </div>
        <div class="userAvatar">БМ</div>
      </div>
      <div class="exit">
        <img src="/icons/log-out.svg" alt="logout"/>
        <div class="exitText">Выйти</div>
      </div>

      <div class="language">
        <img class="flag" src="/icons/russia.svg" alt="Russia">
        <select class="languageSelect">
          <option>Русский</option>
        </select>
      </div>

      <div class="version">
        Версия 1.02
      </div>
    </menu>
    <div class="main">
      <header>
        <div>Учебные сессии</div>
        <div class="icon-bar">
          <div class="search-wrapper">
            <img src="/icons/search.svg" alt="Поиск" class="search-icon">
            <input 
              type="text" 
              placeholder="Поиск" 
              class="search-input"
              v-model="searchQuery"
              @input="handleSearch"
            >
          </div>
          <img src="/icons/hopper.svg" alt="Filter" class="icon">
          <img src="/icons/3lines.svg" alt="Menu" class="icon">
          <button class="submit-btn">Создать</button>
        </div>
      </header>
      <div class="main-table">
        <table>
          <thead>
          <tr>
            <th>
            <span class="th-flex">
              Дата и время
              <img 
                :src="sortDirection === 'asc' ? '/icons/down-arrow.svg' : '/icons/up-arrow.svg'" 
                alt="Сортировка" 
                class="th-icon"
                @click="toggleSort"
                :class="{ 'sort-active': true }"
              >
            </span>
            </th>
            <th>Статус</th>
            <th>Название учебного модуля</th>
            <th>Тип сессии</th>
            <th>Комната</th>
            <th>Группа</th>
          </tr>
          </thead>
          <tbody>
          <tr v-for="session in paginatedSessions" :key="session.id">
            <td>{{ formatDateTime(session.start) }}</td>
            <td>
            <span :class="getStatusClass(session.status.name)">
              {{ getStatusText(session.status.name) }}
            </span>
            </td>
            <td>{{ session.module }}</td>
            <td>{{ getTypeText(session.type.name) }}</td>
            <td>{{ getRoomNames(session.rooms) }}</td>
            <td>{{ getGroupNames(session.groups) }}</td>
          </tr>
          </tbody>

          <!-- ПАГИНАЦИЯ -->
          <tfoot>
          <tr>
            <td colspan="6">
              <div class="pagination">
                <button
                    class="page-btn prev"
                    @click="goToPage(currentPage - 1)"
                    :disabled="currentPage === 1"
                >
                  <img src="/icons/arrowLeft.svg" alt="Назад">
                </button>

                <button
                    v-for="page in visiblePages"
                    :key="page"
                    :class="['page-btn', { active: page === currentPage }]"
                    @click="goToPage(page)"
                >
                  {{ page }}
                </button>

                <button
                    class="page-btn next"
                    @click="goToPage(currentPage + 1)"
                    :disabled="currentPage === totalPages"
                >
                  <img src="/icons/arrowRight.svg" alt="Вперед">
                </button>
              </div>
            </td>
          </tr>
          </tfoot>
        </table>
      </div>

    </div>
  </div>
</template>

<style>
/* http://meyerweb.com/eric/tools/css/reset/
   v2.0 | 20110126
   License: none (public domain)
*/

html, body, div, span, applet, object, iframe,
h1, h2, h3, h4, h5, h6, p, blockquote, pre,
a, abbr, acronym, address, big, cite, code,
del, dfn, em, img, ins, kbd, q, s, samp,
small, strike, strong, sub, sup, tt, var,
b, u, i, center,
dl, dt, dd, ol, ul, li,
fieldset, form, label, legend,
table, caption, tbody, tfoot, thead, tr, th, td,
article, aside, canvas, details, embed,
figure, figcaption, footer, header, hgroup,
menu, nav, output, ruby, section, summary,
time, mark, audio, video {
    margin: 0;
    padding: 0;
    border: 0;
    font-size: 100%;
    font: inherit;
    vertical-align: baseline;
}
/* HTML5 display-role reset for older browsers */
article, aside, details, figcaption, figure,
footer, header, hgroup, menu, nav, section {
    display: block;
}
body {
    line-height: 1;
}
ol, ul {
    list-style: none;
}
blockquote, q {
    quotes: none;
}
blockquote:before, blockquote:after,
q:before, q:after {
    content: '';
    content: none;
}
table {
    border-collapse: collapse;
    border-spacing: 0;
}

/* style.css */

menu {
    width: 274px;
    height: 100vh;
    background: #fff;
    margin-right: 8px;
    display: flex;
    flex-direction: column;
    position: fixed;
    left: 0;
    top: 0;
    overflow-y: hidden;
}

* {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
}

body {
    font-family: "Manrope", sans-serif;
    background: #F4F4F8;
    color: #2F3144;
}

.logo {
    display: flex;
    align-items: center;
    padding: 19px 41px;
}

.logoImg {
    width: 46px;
    height: 46px;
    margin-right: 4px;
}

.logoText {
    font-size: 24px;
    font-weight: 800;
    line-height: 28px;
    letter-spacing: 0px;
}

.tabs {
    padding: 0 12px;
    display: flex;
    flex-direction: column;
}

.tab {
    display: flex;
    align-items: center;
    padding: 12px;
    border-radius: 16px;
    margin-bottom: 4px;
    cursor: pointer;
    transition: background 0.2s;
}

.tab:hover {
    background: #F4F4F4;
}

.tabIcon {
    width: 24px;
    height: 24px;
    margin-right: 12px;
    filter: grayscale(100%) opacity(0.7);
}

.tabText {
    font-size: 15px;
    line-height: 24px;
    letter-spacing: 0;
    font-weight: 800;
}

/* Highlighted tab (Учебные сессии) */
.tab:nth-child(2) {
    background: #3761F3;
}

.tab:nth-child(2) .tabText {
    color: #fff;
}

.tab:nth-child(2) .tabIcon {
    filter: brightness(0) invert(1);
}

.user {
    display: flex;
    align-items: center;
    cursor: pointer;
    justify-content: space-between;
    padding: 12px;
    border-radius: 20px;
    margin: auto 12px 18px;
    box-shadow: 0 4px 6px #0000001F;
}

.user .userAvatar {
    width: 48px;
    height: 48px;
    border-radius: 48px;
    background: #3761F3;
    color: #fff;
    font-size: 15px;
    font-weight: 800;
    line-height: 24px;
    letter-spacing: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
}

.userInfo {
    display: flex;
    flex-direction: column;
}

.userName {
    font-size: 15px;
    font-weight: 800;
    line-height: 24px;
    letter-spacing: 0;
}

.userStatus {
    font-size: 13px;
    font-weight: 500;
    line-height: 20px;
    letter-spacing: 0;
    color: #2F3144A6;
}

.exit {
    display: flex;
    align-items: center;
    padding: 12px;
    cursor: pointer;
    margin-left: 12px;
    margin-right: 12px;
    border-radius: 20px;
    margin-bottom: 4px;
    transition: background 0.2s;
}

.exit:hover {
    background: #f0f0f0;
}

.exit img {
    width: 24px;
    height: 24px;
    margin-right: 12px;
    filter: grayscale(100%) opacity(0.7);
}

.exitText {
    font-size: 15px;
    font-weight: 800;
    line-height: 24px;
    letter-spacing: 0;
}

/* Language select styling */
.language {
    display: flex;
    align-items: center;
    margin-right: 12px;
    margin-left: 12px;
    padding: 12px;
    border-radius: 12px;
    cursor: pointer;
    position: relative;
    border: 1px solid #E0E0E0;
    margin-bottom: 18px;
    transition: background 0.2s;
}

.language:hover {
    background: #f0f0f0;
}

.flag {
    width: 24px;
    height: 24px;
    margin-right: 12px;
    border-radius: 4px;
}

.languageSelect {
    font-size: 15px;
    font-weight: 600;
    background: transparent;
    letter-spacing: 0;
    border: none;
    appearance: none;
    cursor: pointer;
    flex: 1;
    padding: 0;
    color: inherit;
}

.language::after {
    content: '';
    background: url('/icons/down-arrow.svg') no-repeat center;
    width: 20px;
    height: 20px;
    position: absolute;
    right: 15px;
    filter: grayscale(100%) opacity(0.7);
}

.version {
    font-size: 13px;
    font-weight: 500;
    line-height: 20px;
    letter-spacing: 0;
    color: #2F3144A6;
    text-align: left;
    margin-left: 12px;
    margin-bottom: 19px;
}

/*MAIN*/

.main {
    background-color: #ffffff;
    min-height: 100vh;
    margin: 8px 8px 8px 282px;
    border-radius: 12px;
    padding-bottom: 24px;
}

.main {
    color: #2F3144;
    padding: 24px;
    background: #f8f9fb;
    height: 100vh;
}

header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 16px;
}

header > div:first-child {
    font-size: 24px;
    font-weight: 800;
    line-height: 28px;
    letter-spacing: 0;
}

.icon-bar {
    display: flex;
    align-items: center;
    gap: 14px;
}

.search-wrapper {
    position: relative;
    display: flex;
    align-items: center;
}

.search-input {
    padding: 10px 8px 10px 48px;
    border: 1px solid #ccc;
    border-radius: 8px;
    outline: none;
    font-weight: 500;
    font-size: 13px;
    line-height: 20px;
    width: 260px;
}

.search-icon {
    position: absolute;
    left: 10px;
    top: 10px;
    width: 24px;
    height: 24px;
    pointer-events: none;
    opacity: 0.6;
}

.icon {
    width: 44px;
    height: 44px;
    cursor: pointer;
    border-radius: 12px;
    background-color: #F4F4F4;
    padding: 10px;
}

.submit-btn {
    background-color: #3761F3;
    font-family: "Manrope", sans-serif;
    border: none;
    border-radius: 12px;
    padding: 10px 16px;
    color: #fff;
    font-size: 15px;
    line-height: 24px;
    letter-spacing: 0;
    font-weight: 800;
    cursor: pointer;
    transition: background 0.2s;
}

.submit-btn:hover {
    background-color: #3756d6;
}

.main-table {
  padding-bottom: 24px;
}

table {
    width: 100%;
    border-collapse: separate;
    border-spacing: 0;
    background: #fff;
    border-radius: 12px;
    overflow: hidden;
    border: 1px solid #E8EAEC;
}

table th, table td {
    padding: 12px 16px;
    border-bottom: 1px solid #E8EAEC;
    text-align: left;
    font-weight: 800;
    font-size: 17px;
    line-height: 28px;
    letter-spacing: 0;
}

/* Фиксированная ширина для первого столбца */
table th:first-child,
table td:first-child {
    width: 206px;
    min-width: 206px;
    max-width: 206px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
}

/* Фиксированная ширина для столбца "Тип сессии" (4-й столбец) */
table th:nth-child(4),
table td:nth-child(4) {
    width: 146px;
    min-width: 146px;
    max-width: 146px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
}

/* Максимальная ширина для столбца "Название учебного модуля" (3-й столбец) */
table th:nth-child(3),
table td:nth-child(3) {
    min-width: 400px;
    max-width: 400px;
    word-wrap: break-word;
    overflow-wrap: break-word;
    hyphens: auto;
}

table th {
    background-color: #F4F4F4;
}

table td {
    font-weight: 500;
    font-size: 15px;
    line-height: 24px;
    padding: 22px 16px;
    word-wrap: break-word;
    overflow-wrap: break-word;
    hyphens: auto;
}

/* Стили для остальных столбцов */
table th:not(:first-child),
table td:not(:first-child) {
    word-wrap: break-word;
    overflow-wrap: break-word;
    hyphens: auto;
}

table tr:hover td {
    background-color: #F4F4F4;
}

table tbody tr:nth-child(even) {
    background-color: #F4F4F4;
}

/* Заголовок с иконкой */
.th-flex {
    display: inline-flex;
    align-items: center;
    gap: 10px;
    white-space: nowrap;
}

.th-icon {
    width: 24px;
    height: 24px;
    cursor: pointer;
    transition: opacity 0.2s;
}

.th-icon:hover {
    opacity: 0.8;
}

.th-icon.sort-active {
    cursor: pointer;
}

.th-icon.sort-active:hover {
    opacity: 0.8;
}

/* Статусы */
.status {
    display: inline-block;
    padding: 4px 12px;
    border-radius: 43px;
    font-size: 13px;
    font-weight: 500;
    letter-spacing: 0;
    line-height: 20px;
    white-space: nowrap;
}

.status.planned {
    background-color: #AFBFF5;
}

.status.progress {
    background-color: #FFDAA1;
}

.status.done {
    background-color: #91C893;
}

.status.canceled {
    background-color: #FFB3B3;
}

thead th,
tfoot td {
    background-color: #F5F7F9 !important;
}

tfoot tr td{
    padding: 11px 16px;
    border: none;
}

.pagination {
    display: flex;
    align-items: center;
    gap: 8px;
    justify-content: flex-start;
}

.page-btn {
    font-family: "Manrope", sans-serif;
    background: #fff;
    border: none;
    padding: 10px;
    border-radius: 8px;
    cursor: pointer;
    font-size: 15px;
    font-weight: 500;
    line-height: 24px;
    width: 30px;
    height: 30px;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: all 0.2s;
}

.page-btn img {
    width: 18px;
    height: 18px;
}

.page-btn.active {
    color: #3761F3;
    border: 1px solid #3761F3;
}

.page-btn:hover:not(:disabled) {
    background: #E0E0E0;
}

.page-btn:disabled {
    opacity: 0.5;
    cursor: not-allowed;
}

.page-btn.prev:disabled,
.page-btn.next:disabled {
    opacity: 0.3;
}
</style>
