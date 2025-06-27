<template>
  <v-card>
    <v-card-title class="justify-center d-flex mb-4">
      Отчет по мероприятиям
    </v-card-title>
      <v-spacer></v-spacer>
      <v-expansion-panels class="panel">
        <v-expansion-panel>
          <v-expansion-panel-title>Фильтры</v-expansion-panel-title>
          <v-expansion-panel-text>
            <div class="filters">
              <v-autocomplete
                v-model="filters.selected.events"
                :items="filters.value.events"
                item-title="VALUE"
                item-value="ID"
                label="Мероприятие"
                single-line
                hide-details
                variant="outlined"
                multiple
                chips
                clearable
              >
                <template v-slot:prepend-item>
                  <v-list-item>
                    <v-list-item-content>
                      <v-list-item-title>
                        <v-checkbox label="Выбрать всех пользователей" v-model="filters.selectAll.events" @change="toggleSelectAll" /> </v-list-item-title>
                    </v-list-item-content>
                  </v-list-item>
                </template>
              </v-autocomplete>
              <v-autocomplete
                v-model="filters.selected.companies"
                :items="filters.value.companies"
                item-title="TITLE"
                item-value="ID"
                label="Компания"
                single-line
                hide-details
                variant="outlined"
                multiple
                chips
                clearable
              >
                <template v-slot:prepend-item>
                  <v-list-item>
                    <v-list-item-content>
                      <v-list-item-title>
                        <v-checkbox label="Выбрать всех пользователей" v-model="filters.selectAll.companies" @change="toggleSelectAll" /> </v-list-item-title>
                    </v-list-item-content>
                  </v-list-item>
                </template>
              </v-autocomplete>
              <v-autocomplete
                v-model="filters.selected.statuses"
                :items="filters.value.statuses"
                item-title="VALUE"
                item-value="ID"
                label="Статус участия"
                single-line
                hide-details
                variant="outlined"
                multiple
                chips
                clearable
              >
                <template v-slot:prepend-item>
                  <v-list-item>
                    <v-list-item-content>
                      <v-list-item-title>
                        <v-checkbox label="Выбрать всех пользователей" v-model="filters.selectAll.statuses" @change="toggleSelectAll" /> </v-list-item-title>
                    </v-list-item-content>
                  </v-list-item>
                </template>
              </v-autocomplete>
              <v-autocomplete
                v-model="filters.selected.years"
                :items="filters.value.years"
                label="Год"
                single-line
                hide-details
                variant="outlined"
                multiple
                chips
                clearable
              >
                <template v-slot:prepend-item>
                  <v-list-item>
                    <v-list-item-content>
                      <v-list-item-title>
                        <v-checkbox label="Выбрать всех пользователей" v-model="filters.selectAll.yeas" @change="toggleSelectAll" /> </v-list-item-title>
                    </v-list-item-content>
                  </v-list-item>
                </template>
              </v-autocomplete>
            </div>
          </v-expansion-panel-text>
        </v-expansion-panel>
      </v-expansion-panels>
      <div class="buttons">
        <v-btn color="primary">отключить фильтры</v-btn>
        <v-btn color="info" @click="getData()">получить данные</v-btn>
      </div>
    <v-data-table
      :headers="headers"
      :items="items"
      :search="search"
      :items-per-page="10"
      class="elevation-1"
    >
      <template v-for="header in allHeaders" :key="header.key" v-slot:[`item.${header.key}`]="{ item }">
        <template v-if="header.key.includes('_status')">
          <v-chip v-if="item[header.key]" :color="getStatusColor(item[header.key])" small>
            {{ item[header.key] }}
          </v-chip>
        </template>
        <template v-else-if="Number.isInteger(item[header.key]) || parseInt(header.key)">
          {{ formatNumber(item[header.key]) }}
        </template>
        <template v-else>
          {{ item[header.key] }}
        </template>
      </template>
      <template v-slot:body.append="{  }">
        <tr>
          <td>Итого:</td>
          <td></td>
          <template v-for="(header, i) in Object.keys(totalRow).length">
                  <td colspan="2">
                    <span class="center">{{Object.values(totalRow)[i]}}</span>
                  </td>
          </template>
        </tr>
      </template>
    </v-data-table>
    <v-data-table
      :headers="headersStatuses"
      :items="groupedDataWithTotals"
      :items-per-page="10"
      class="elevation-1"
    ></v-data-table>
  </v-card>
</template>

<script setup>
import { ref, onMounted, watch, computed } from 'vue'
import { callApi } from '../functions/callApi';
import moment from 'moment';

const isLoading = ref(true);
const items = ref([]);
const headers = ref([]);
const statuses = ref([]);
const groupBy = [{ key: 'EVENT', order: 'asc' }];
const sortBy = [{ key: 'UF_CRM_1750742107', order: 'asc' }];
const groupByStatuses = [{ key: 'DATE_CREATE', order: 'desc' }];

const headersStatuses = [
  { title: 'Год', key: 'DATE_CREATE'},
  { title: 'Сумма', key: 'OPPORTUNITY'},
  { title: 'Статус участия', key: 'UF_CRM_1750742107'},
  { title: 'Количество компаний', key: 'count'},
];

function toggleSelectAll(){
  console.log(filters.value);

  for (let i = 0; i < Object.keys(filters.value.selectAll).length; i++) {
    console.log(Object.values(filters.value.value)[i]);
/*
    if (Object.keys(filters.value.selectAll)[i] === true) {
        console.log(i);
    }
*/
  }
}

function findEventName(id, fields) {
  if(id === "0"){
    return null;
  }

  let foundItem = fields.find(item => item.ID === id);
  if(foundItem.VALUE){
    return foundItem.VALUE;
  }else if(foundItem.TITLE){
    return foundItem.TITLE;
  }else{
    return null;
  }
}

const filtredItems = computed(() => {
return items.value.filter(item => {
        return (
          (filters.value.selected.events.includes(item.EVENT)) &&
          (filters.value.selected.companies.includes(item.COMPANY_ID)) &&
          (Object.keys(obj).some(key => 
    key.endsWith('_status') && 
    (filters.value.selected)
  )) &&
          (filters.value.selected.years.includes(item.EVENT))
)
      });
});

const allHeaders = computed(() => {
  const result = [];
  headers.value.forEach(header => {
    if (header.children) {
      result.push(...header.children);
    } else {
      result.push(header);
    }
  });
  return result;
});


function mergedData(data) {
  const result = {};
  
  data.forEach(item => {
    // Создаем уникальный ключ из COMPANY_ID и EVENT
    const key = `${item.COMPANY_ID}_${item.EVENT}`;
    
    if (!result[key]) {
      // Если такого ключа еще нет, создаем новую запись
      result[key] = {
        COMPANY_ID: item.COMPANY_ID,
        EVENT: item.EVENT,
        years: {}
      };
    }
    
    // Перебираем все года в объекте (2023, 2024 и т.д.)
    for (const [prop, value] of Object.entries(item)) {
      if (/^\d{4}$/.test(prop)) {
        // Это свойство - год (например "2023")
        const year = prop;
        const statusKey = `${year}_status`;
        const status = item[statusKey];
        
        if (!result[key].years[year]) {
          result[key].years[year] = {
            sum: 0,
            status: status
          };
        }
        
        // Суммируем значение, преобразуя строку в число
        const numericValue = parseFloat(value) || 0;
        result[key].years[year].sum += numericValue;
      }
    }
  });
  
  // Преобразуем результат в более удобный формат
  return Object.values(result).map(entry => {
    const formattedEntry = {
      COMPANY_ID: entry.COMPANY_ID,
      EVENT: entry.EVENT
    };
    
    // Добавляем года с суммами и статусами
    for (const [year, data] of Object.entries(entry.years)) {
      formattedEntry[year] = data.sum.toString();
      formattedEntry[`${year}_status`] = data.status;
    }
    
    return formattedEntry;
  });
}

const processedItems = computed(() => {
  const result = [];
  const eventsMap = {};
  
  // Группируем по мероприятиям
  items.value.forEach(item => {
    if (!eventsMap[item.EVENT]) {
      eventsMap[item.EVENT] = {};
    }
    
    if (!eventsMap[item.EVENT][item.DATE_CREATE]) {
      eventsMap[item.EVENT][item.DATE_CREATE] = [];
    }
    
    eventsMap[item.EVENT][item.DATE_CREATE].push(item);
  });
  
  // Добавляем данные и итоги
  Object.keys(eventsMap).forEach(eventName => {
    const years = Object.keys(eventsMap[eventName]);
    
    years.forEach(year => {
      // Добавляем записи за год
      result.push(...eventsMap[eventName][year]);
      
      // Добавляем итог за год
      const yearTotal = {
        EVENT: eventName,
        DATE_CREATE: year,
        OPPORTUNITY: eventsMap[eventName][year].reduce((sum, item) => sum + Number(item.OPPORTUNITY), 0),
        isYearTotal: true
      };
      result.push(yearTotal);
    });
    
    // Добавляем итог за все годы
    const allYearsTotal = {
      EVENT: eventName,
      DATE_CREATE: 'Все годы',
      OPPORTUNITY: years.reduce((sum, year) => sum + eventsMap[eventName][year].reduce((s, item) => s + Number(item.OPPORTUNITY), 0), 0),
      isTotal: true
    };
    result.push(allYearsTotal);
  });
  
  return result;
});

const groupedDataWithTotals = computed(() => {
  const grouped = {};
  const yearTotals = {};
  
  items.value.forEach(item => {
    const key = `${item.UF_CRM_1750742107}_${item.DATE_CREATE}`;
    
    if (!grouped[key]) {
      grouped[key] = {
        UF_CRM_1750742107: item.UF_CRM_1750742107,
        DATE_CREATE: item.DATE_CREATE,
        OPPORTUNITY: 0,
        count: 0
      };
    }
    
    grouped[key].OPPORTUNITY += Number(item.OPPORTUNITY);
    grouped[key].count += 1;
    
    // Считаем итоги по годам
    if (!yearTotals[item.DATE_CREATE]) {
      yearTotals[item.DATE_CREATE] = {
        DATE_CREATE: item.DATE_CREATE,
        OPPORTUNITY: 0,
        count: 0
      };
    }
    yearTotals[item.DATE_CREATE].OPPORTUNITY += Number(item.OPPORTUNITY);
    yearTotals[item.DATE_CREATE].count += 1;
  });
  
  const result = Object.values(grouped);
  
  // Добавляем итоги по годам
  Object.values(yearTotals).forEach(year => {
    result.push({
      ...year,
      isYearTotal: true
    });
  });
  
  // Добавляем общий итог
  const totalAllYears = {
    DATE_CREATE: 'Все годы',
    OPPORTUNITY: Object.values(yearTotals).reduce((sum, year) => sum + year.OPPORTUNITY, 0),
    count: Object.values(yearTotals).reduce((sum, year) => sum + year.count, 0),
    isTotal: true
  };
  result.push(totalAllYears);
  
  return result;
});
  // Создаем объект для итоговой строки
const totalRow = ref({
  total: 0,
});

const filters = ref({

value: {
  'companies': '',
  'events': '',
  'years': '',
  'statuses': '',
},

selected: {
  'companies': [],
  'events': [],
  'years': [],
  'statuses': [],
},

selectAll: {
  'companies': false,
  'events': false,
  'years': false,
  'statuses': false,
}
});

let fields = [];

async function getFilters(){
    fields = await callApi("crm.deal.fields", {}, ["OPPORTUNITY", "UF_CRM_1750742107", "UF_CRM_1750742743", "DATE_CREATE"], null, null, null);
    filters.value.value.years = await new Promise((resolve) => {
      BX24.callMethod("crm.deal.list", {
        order: {"ID": "ASC"},
        select: ["DATE_CREATE"]
      }, (res) => {
        if (res.data()) {
          const years = [];
          const currentYear = moment().year();
          const startYear = moment(res.data()[0].DATE_CREATE).format('YYYY');
          for (let year = startYear; year <= currentYear; year++) {
            years.push(year);
          }
          resolve(years);
        }
      });
    });

    filters.value.value.companies = await callApi("crm.company.list", {}, [], null, null, null);
    filters.value.value.events = fields.UF_CRM_1750742743.items;
    filters.value.value.statuses = fields.UF_CRM_1750742107.items;
}

onMounted(async () => {
  await getFilters();
  isLoading.value = false;
});

const getData = async() => {//, "COMPANY_ID": filters.value.selected.companies
  let localItems = await callApi("crm.deal.list", {"!UF_CRM_1750742743": "NULL", "!UF_CRM_1750742107": "NULL", "DATE_CREATE": "2025%"}, ["OPPORTUNITY", "UF_CRM_1750742107", "UF_CRM_1750742743", "DATE_CREATE", "COMPANY_ID"], null, null, null);
  const uniqueCompanies = Array.from(new Set(localItems.map(item => item.COMPANY_ID)));

  const companies = await callApi("crm.company.list", {"ID": [...uniqueCompanies]}, [], null, null, null);

  localItems = localItems.map(item => ({
    ...item,
    EVENT: findEventName(item.UF_CRM_1750742743, fields.UF_CRM_1750742743.items),
    COMPANY_ID: findEventName(item.COMPANY_ID, companies),
    [moment(item.DATE_CREATE).format('YYYY')]: (+item.OPPORTUNITY).toFixed(0),
    [moment(item.DATE_CREATE).format('YYYY') + '_status']: findEventName(item.UF_CRM_1750742107, fields.UF_CRM_1750742107.items),
  }));

const uniqueYears = localItems
  .map(item => moment(item.DATE_CREATE).format('YYYY')) // Получаем массив годов
  .filter((year, index, self) => self.indexOf(year) === index); // Оставляем только уникальные года
  localItems = mergedData(localItems);

  filters.value.value.years = uniqueYears;

  filters.value.value.companies = companies;
  
  console.log(filters.value);
// Создаем массив объектов на основе уникальных годов
const yearObjects = uniqueYears.map(year => ({
  title: year,
  key: year,
  align: 'center',
  children: [
    {title: 'Статус участия', key: year + '_status'},
    {title: 'Сумма', key: year},
  ]
}));

console.log(yearObjects);

console.log(localItems);

const yearSums = uniqueYears.map(year => ({
  [year]: 0
}));

  const result = {
    grandTotal: 0
  };

  // Рассчитываем суммы по каждому году
  localItems.forEach(item => {
    uniqueYears.forEach(year => {
      if (item[year]) {
        const value = parseInt(item[year]) || 0;
        if(yearSums[year]){
          yearSums[year] += value;
        }else{
          yearSums[year] = value;
        }
      }
    });
  });

  console.log(uniqueYears);

  localItems.map(item => {
    const years = {};
    item.total = 0;
    // Перебираем все свойства объекта
    Object.keys(item).forEach(key => {
      // Проверяем, является ли ключ годом (4 цифры)
      if (/^\d{4}$/.test(key)) {
        const value = parseFloat(item[key]) || 0;
        years[key] = value;
        console.log(item.total);
        item.total += value;
        totalRow.value.total += value;
      }
    });
    //console.log(totalRow.value.total);
    //totalRow.value.total = formatNumber(totalRow.value.total);
  });

  console.log(result);

  // Добавляем суммы и статусы в итоговую строку
  uniqueYears.forEach(year => {
    if (yearSums[year] > 0) {
      totalRow.value[year] = formatNumber(yearSums[year]);
    } else {
      totalRow.value[year] = 0;
    }
  });
  totalRow.value.total = formatNumber(totalRow.value.total);
  console.log(totalRow.value);

  headers.value = [
    { title: 'Мероприятие', key: 'EVENT', with: '25%'},
    { title: 'Компания', key: 'COMPANY_ID'},
  ].concat(yearObjects).concat({ title: 'Сумма', key: 'total', with: '25%'});

  items.value = localItems;
  isLoading.value = false;
};

  const getStatusColor = (status) => {
    const colors = {
      'Спонсор': 'primary',
      'Генеральный партнер': 'success',
      'Стратегический партнер': 'info',
      'Участник': 'warning'
    };
    console.log(status);
    return colors[status] || 'secondary';
  };

const formatNumber = (num) => {
  if(num > 0){
    return new Intl.NumberFormat("ru-RU", { style: "currency", currency: "RUB" }).format(num);
  }
}

</script>

<style lang="sass">

  #app
    margin: 0.75rem

  .v-data-table-footer
    justify-content: center

  .center
    display: flex
    align-items: center
    justify-content: center

  th, td
    border: 1px solid rgba(0, 0, 0, 0.12)

  .filters
    display: grid
    grid-template-columns: 1fr 1fr
    gap: 1rem
    margin-bottom: 1rem

  .v-input.v-text-field
    display: block !important

  .v-input__details
    display: none

  .panel
    margin-bottom: 1rem

  .buttons
    width: 100%
    display: flex
    align-items: center
    justify-content: center
    gap: 1rem
    margin-bottom: 1rem

</style>