<template>
  <q-page class="flex flex-center">
    <div class="q-pa-md">
      <q-table
        title="Список действий"
        :data="data"
        :columns="columns"
        row-key="id"
        :filter="search"
        :pagination.sync="myPagination"
      >
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th auto-width />
            <q-th
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
            >
              {{ col.label }}
            </q-th>
            <q-th>Редактировать</q-th>
            <q-th>Удалить</q-th>
          </q-tr>
        </template>
        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td auto-width>
              <q-btn size="sm"
                     color="primary"
                     round
                     dense
                     @click="props.expand = !props.expand"
                     :icon="props.expand ? 'remove' : 'add'" />
            </q-td>
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
            >
              {{ col.value }}
            </q-td>
            <q-td class="text-center">
              <q-btn
                size="sm"
                round
                color="primary"
                icon="edit"
                @click="edit()" />
            </q-td>
            <q-td class="text-center">
              <q-btn
                size="sm"
                round
                color="primary"
                icon="remove"
                @click="remove(props.row.id, data)" />
            </q-td>
          </q-tr>
          <q-tr
            v-for="(actionObj, index) in props.row.actions"
            :key="index"
            v-show="props.expand"
            :props="props" >
            <q-td>
              <q-btn
                size="sm"
                round
                color="primary"
                icon="arrow_upward"
                @click="moveToTheTop(props.row, actionObj, index)" />
            </q-td>
            <q-td colspan="6">
              <ul>
                <li
                  v-for="(value, index) in actionObj"
                  :key="index">{{fields[index]}}: {{value}}
                </li>
              </ul>
            </q-td>
            <q-td class="text-center">
              <q-btn
                size="sm"
                round
                color="primary"
                icon="edit"
                @click="edit(index, props.row)" />
            </q-td>
            <q-td class="text-center">
              <q-btn
                size="sm"
                round
                color="primary"
                icon="remove"
                @click="remove(index, props.row)" />
            </q-td>
          </q-tr>
        </template>
        <template v-slot:top-right>
          <q-input borderless dense debounce="300" v-model="search" placeholder="Search">
            <template v-slot:append>
              <q-icon name="search" />
            </template>
          </q-input>
        </template>
        <template v-slot:bottom>
          NEXT
        </template>
      </q-table>
    </div>

    <div class="q-pa-md">
    <q-btn-dropdown color="primary" label="Выберите действие:">
      <q-list>
        <q-item clickable v-close-popup @click="onChangeAction('CREATE')">
          <q-item-section>
            <q-item-label>CREATE</q-item-label>
          </q-item-section>
        </q-item>

        <q-item clickable v-close-popup @click="onChangeAction('SET')">
          <q-item-section>
            <q-item-label>SET</q-item-label>
          </q-item-section>
        </q-item>

        <q-item clickable v-close-popup @click="onChangeAction('BITWISE')">
          <q-item-section>
            <q-item-label>BITWISE</q-item-label>
          </q-item-section>
        </q-item>

        <q-item clickable v-close-popup @click="onChangeAction('REPLACE')">
          <q-item-section>
            <q-item-label>REPLACE</q-item-label>
          </q-item-section>
        </q-item>
      </q-list>
    </q-btn-dropdown>
      <Form
        :action=action
        :allVariables=variables
        :strVariables=strVariables
        :data=data
        :editing=editing
        :editingArr=editingArr
        @add="add"
        @create="create"
       />
    </div>
  </q-page>
</template>

<script>
import Form from '../components/Form.vue';

export default {
  name: 'PageIndex',
  components: {
    Form,
  },
  data() {
    return {
      myPagination: {
        rowsPerPage: 0,
      },
      search: '',
      editing: false,
      editingArr: [],
      columns: [
        {
          name: 'action', align: 'center', label: 'Действие', field: 'action',
        },
        {
          name: 'name', align: 'center', label: 'Наименование переменной', field: 'name',
        },
        {
          name: 'type', align: 'center', label: 'Тип', field: 'type',
        },
        {
          name: 'size', align: 'center', label: 'Размер', field: 'size',
        },
        {
          name: 'value', align: 'center', label: 'Значение по умолчанию', field: 'value',
        },
        {
          name: 'description', align: 'center', label: 'Описание', field: 'description',
        },
      ],
      data: [],
      action: '',
      variables: [],
      strVariables: [],
      removeConf: false,
      confirmVar: false,
      fields: {
        action: 'Действие',
        variableName: 'Имя переменной',
        newValue: 'Новое значение',
        description: 'Описание',
        oldStr: 'Заменяемая строка',
        newStr: 'Заменяющая строка',
        variableResult: 'Результирующая переменная',
        operation: 'Побитовая операция',
        mask: 'Маска',
      },
    };
  },
  methods: {
    onChangeAction(action) {
      this.action = action;
    },
    create(data) {
      this.data.push(data);
      this.getAllVariables();
    },
    add(variable, data) {
      const index = this.data.findIndex((item) => item.name === variable);
      this.data[index].actions.push(data);
      this.alert('Действие добавлено!');
    },
    getAllVariables() {
      const createdVariables = this.data.filter((item) => item.action === 'CREATE');
      this.variables = createdVariables.map((item) => item.name);
      this.strVariables = createdVariables.filter((item) => item.type === 'string').map((item) => item.name);
    },
    moveToTheTop(obj, action, ind) {
      if (ind === 0) {
        obj.actions.shift();
        obj.actions.push(action);
      } else {
        obj.actions.splice(ind - 1, 2, obj.actions[ind], obj.actions[ind - 1]);
      }
    },
    edit(ind, data) {
      this.editing = true;
      this.editingArr = [ind, data];
      this.action = data.actions[ind].action;
    },
    remove(ind, data) {
      let message = '';
      let updateVars;
      if (data.actions) {
        message = 'Вы точно хотите удалить это действие?';
        updateVars = false;
        this.confirmDelete(ind, message, data.actions, updateVars);
      } else {
        const deleteIndex = this.data.findIndex((item) => item.id === ind);
        message = 'Если Вы удалите CREATE, то все действия с этой переменной автоматически удалятся. Вы точно хотите это сделать?';
        updateVars = true;
        this.confirmDelete(deleteIndex, message, data, updateVars);
      }
    },
    confirmDelete(ind, message, data, update) {
      this.$q.dialog({
        title: 'Внимание',
        message,
        cancel: true,
        persistent: true,
      }).onOk(() => {
        data.splice(ind, 1);
        if (update) {
          this.getAllVariables();
        }
      });
    },
    alert(message) {
      this.$q.dialog({
        message,
      });
    },
  },


};
</script>
