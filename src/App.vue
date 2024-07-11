<template>
  <v-app class="ma-8">
    <v-main>
      <div class="d-flex ga-8">
        <div class="actions">
            <v-text-field
                    v-model="searchText"
                    color="primary"
                    label="Поиск"
                    variant="underlined"
            ></v-text-field>
          <div class="d-flex flex-column ga-2">
            <template v-for="item in getAllProperties" :key="item.key">
              <v-card
                class="mx-auto"
                width="200"
                height="50"
                link
                @click="chosenItem(item)"
              >
                <div class="card_slot-wrap d-flex flex-column mx-2">
                    <div class="text-body-2 font-weight-medium">{{item.label}}</div>
                    <div class="text-caption">{{item.type}}</div>
                </div>
              </v-card>
            </template>
          </div>
        </div>
          <div class="choseItems">
              <div class="choseActions d-flex align-center ga-2">
                  <v-btn icon="$collapse" variant="plain" @click="forToPrew"></v-btn>
                  <v-btn icon="$expand" variant="plain" @click="forToNext"></v-btn>
                  <v-btn variant="text" @click="rmChosenItem">rm</v-btn>
                  <v-text-field
                      v-model="colCount"
                      min-width="90"
                      color="primary"
                      label="Cols"
                      type="number"
                      variant="underlined"
                      @input="inputCol"
                  ></v-text-field>
              </div>
              <div v-if="choseItems.length" class="d-flex flex-column ga-2">
                  <div v-for="(choseItem, idx) in choseItems" :key="idx">
                      <v-card
                          class="mx-auto"
                          height="50"
                          link
                          :variant="selectKeyChosenItem === choseItem.key ? 'tonal' : undefined"
                          @click.stop="selectItem(choseItem.key)"
                      >
                          <div class="card_slot-wrap d-flex flex-column mx-2">
                              <div class="text-body-2 font-weight-medium">{{choseItem.label}}</div>
                              <div class="text-caption">{{choseItem.type}}</div>
                          </div>
                      </v-card>
                  </div>
              </div>
              <div v-else>empty for now</div>
          </div>
        <div class="preview w-100">
            <v-container>
                <v-row v-for="(row, rowKey) in mappingPreview" :key="rowKey">
                    <v-col v-for="(col) in row" :key="col.key" :cols="col.col">
                        <div>
                          <component
                            :is="col.component"
                            v-bind="col.props"
                            v-model="col.model"
                            :label="col.label"
                          ></component>
                        </div>
                    </v-col>
                </v-row>
            </v-container>
        </div>
      </div>
    </v-main>
  </v-app>
</template>

<script setup>
import {computed, markRaw, ref, shallowRef} from "vue";
import {VTextField} from "vuetify/components/VTextField";
import { VSelect} from "vuetify/components/VSelect";
import {VDateInput} from "vuetify/labs/VDateInput";


const searchText = ref('')
const allProperties = ref([
  {
    type: 'date',
    key: 'dateDoc',
    label: 'Дата документа',
      component: shallowRef(VDateInput),
      props: {
        model: '',
        variant: "outlined",
        prependIcon: ''
      },
      col: 6
  },
  {
    type: 'number',
    key: 'numberDoc',
    label: 'Номер документа',
      component: shallowRef(VTextField),
      props: {
        color: "primary",
        model: '',
        variant: "outlined",
        type: 'number'
      },
      col: 6
  },
  {
    type: 'date',
    key: 'dateCreate',
    label: 'Дата создания документа',
      component: shallowRef(VDateInput),
      props: {
        model: '',
        variant: "outlined",
        prependIcon: ''
      },
      col: 6
  },
  {
    type: 'select',
    key: 'state',
    label: 'Состояние',
      component: shallowRef(VSelect),
      props: {
        items: ['На подписании', 'Подписан', 'Черновик'],
        model: '',
        variant: "outlined",
      },
      col: 6
  },
  {
    type: 'select',
    key: 'partner',
    label: 'Контрагент',
      component: shallowRef(VSelect),
      props: {
        items: ['ООО Ромашка', 'АО Глобал', 'КФХ Коллектив'],
        chips: true,
        multiple: true,
        model: '',
        variant: "outlined",
      },
      col: 6
  },
  {
    type: 'text',
    key: 'theme',
    label: 'Тема',
      component: shallowRef(VTextField),
      props: {
        model: '',
        variant: "outlined",
      },
      col: 6
  },
  {
    type: 'select',
    key: 'coordinating',
    label: 'Согласующий',
      component: shallowRef(VSelect),
      props: {
        items: ['Иванов', 'Петров', 'Сидоров'],
        chips: true,
        multiple: true,
        model: '',
        variant: "outlined",
      },
      col: 6
  },
  {
    type: 'select',
    key: 'creator',
    label: 'Создал',
      component: shallowRef(VSelect),
      props: {
        model: '',
        items: ['Иванов', 'Петров', 'Сидоров'],
        variant: "outlined",
      },
      col: 6
  },
  {
    type: 'select',
    key: 'typeDoc',
    label: 'Тип документа',
      component: shallowRef(VSelect),
      props: {
        model: '',
        items: ['Приказ', 'Распоряжение', 'Договор подряда'],
        variant: "outlined",
      },
      col: 6
  }
])
const getAllProperties = computed(() => {
    if (!searchText.value) return allProperties.value
    else return allProperties.value.filter(i => i.label.toLowerCase().includes(searchText.value.toLocaleLowerCase()))
})

const choseItems = ref([])
const selectKeyChosenItem = ref('')
const colCount = ref()
const chosenItem = (item) => {
  if (choseItems.value.find(i => i.key === item.key)) return
    choseItems.value.push(item)
    colCount.value = ''
  selectKeyChosenItem.value = ''
}
const selectItem = (key) => {
    colCount.value = choseItems.value.find(i => i.key === key).col
    selectKeyChosenItem.value = key
}
const forToPrew = () => {
  const currentIdx = choseItems.value.findIndex(i => i.key === selectKeyChosenItem.value)
  if (currentIdx > 0) {
    const obj = choseItems.value.splice(currentIdx, 1)
    choseItems.value.splice(currentIdx - 1, 0, obj[0])
  }
}
const forToNext = () => {
  const currentIdx = choseItems.value.findIndex(i => i.key === selectKeyChosenItem.value)
  if (currentIdx <= choseItems.value.length) {
    const obj = choseItems.value.splice(currentIdx, 1)
    choseItems.value.splice(currentIdx + 1, 0, obj[0])
  }
}
const rmChosenItem = () => {
    choseItems.value.splice(choseItems.value.findIndex(i => i.key === selectKeyChosenItem.value), 1)
}
const inputCol = () => {
    if (colCount.value > 12) colCount.value = 12
    choseItems.value.find(i => i.key === selectKeyChosenItem.value).col = +colCount.value
}

const mappingPreview = computed(() => {
    if (!choseItems.value.length) return false
    else {
        const rows = {}
        let rowsCounter = 0
        let counterCols = 0
        choseItems.value.forEach(i => {
            if (counterCols + i.col <= 12) {
                counterCols += i.col
              if (rows[rowsCounter]) {
                rows[rowsCounter].push(i)
              }
              else rows[rowsCounter] = [i]
            } else {
              counterCols = i.col
              ++rowsCounter
              rows[rowsCounter] = [i]
            }
        })
        return rows
    }
})
</script>
<style scoped>
.choseActions {
    min-width: 220px;
}
</style>
