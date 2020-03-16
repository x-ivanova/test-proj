<template>
  <q-card class="my-card" v-if="action">
    <div class="q-pa-md">
      <div class="text-h6">{{action}}</div>
      <q-form v-if="action === 'CREATE'"
        @submit="onSubmitCreate"
        @reset="onReset()"
        class="q-gutter-md" >
        <q-input
          id="name"
          v-model="name"
          label="Наименование переменной"
          required
          bottom-slots
          hint="Поле должно содержать только латинские буквы и знак '_'"
          :error="!isValidName"
          error-message="Поле должно содержать только латинские буквы и знак '_'" />
        <q-select
          v-model="type"
          :options="types"
          label="Тип"
          :rules="[ val => val !== null || 'Обязательное поле' ]"
          bottom-slots
          hint="Обязательное поле"
          :disable="editing === true" />
        <q-input
          v-model.number="size"
          type="number"
          label="Размер" />
        <q-input
          v-model="value"
          label="Значение по умолчанию"
          bottom-slots
          hint="Значение должно соответствовать типу переменной"
          :error="!isValidValue(this.type, this.value, this.size)"
          error-message="Значение не соответствует типу переменной." />
        <q-input
          v-model="description"
          filled
          type="textarea"
          label="Описание" />
        <div>
          <q-btn label="Submit" type="submit" color="primary"/>
          <q-btn label="Reset" type="reset" color="primary" flat class="q-ml-sm" />
        </div>
      </q-form>
      <q-form v-if="action === 'SET'"
        @submit="onSubmitSet"
        @reset="onReset"
        class="q-gutter-md" >
        <q-select
          v-model="variableSet"
          :options="allVariables"
          label="Выберите переменную"
          :rules="[ val => val !== null || 'Обязательное поле' ]"
          bottom-slots
          hint="Обязательное поле"
          :disable="editing === true"
        />
        <q-input
          filled
          v-model="valueSet"
          label="Новое значение"
          bottom-slots
          hint="Обязательное поле"
          :error="!isValidValueSet"
          error-message="Значение не соответствует типу переменной либо превышает допустимый размер"
          required />
        <q-input
          v-model="descriptionSet"
          filled
          type="textarea"
          label="Описание" />
        <div>
          <q-btn label="Submit" type="submit" color="primary"/>
          <q-btn label="Reset" type="reset" color="primary" flat class="q-ml-sm" />
        </div>
      </q-form>
      <q-form v-if="action === 'REPLACE'"
              @submit="onSubmitReplace"
              @reset="onReset"
              class="q-gutter-md" >
        <q-select
          v-model="variableReplace"
          :options="strVariables"
          label="Выберите переменную"
          :rules="[ val => val !== null || 'Обязательное поле' ]"
          bottom-slots
          hint="Обязательное поле"
          :disable="editing === true" />
        <q-input
          v-model="oldStr"
          label="Заменяемая строка"
          required
          bottom-slots
          hint="Обязательное поле" />
        <q-input
          v-model="newStr"
          label="Заменяющая строка"
          required
          bottom-slots
          hint="Обязательное поле" />
        <q-select
          v-model="resVariable"
          :options="strVariables"
          label="Результирующая переменная"
          :rules="[ val => val !== null || 'Обязательное поле' ]"
          bottom-slots
          hint="Обязательное поле" />
        <q-input
          v-model="descriptionReplace"
          filled
          type="textarea"
          label="Описание"
        />
        <div>
          <q-btn label="Submit" type="submit" color="primary"/>
          <q-btn label="Reset" type="reset" color="primary" flat class="q-ml-sm" />
        </div>
      </q-form>

      <q-form v-if="action === 'BITWISE'"
              @submit="onSubmitBitwise"
              @reset="onReset"
              class="q-gutter-md" >
        <q-select
          v-model="variableBit"
          :options="allVariables"
          label="Выберите переменную"
          :rules="[ val => val !== null || 'Обязательное поле' ]"
          bottom-slots
          hint="Обязательное поле"
          :disable="editing === true" />
        <q-select
          v-model="typeBit"
          :options="typesBit"
          label="Тип сдвига"
          :rules="[ val => val !== null || 'Обязательное поле' ]"
          bottom-slots
          hint="Обязательное поле" />
        <q-input
          filled
          v-model="mask"
          label="Маска (16-ричный формат)"
          bottom-slots
          hint="Поле должно содержать число в 16-ричный формате"
          :error="!isValidMask"
          error-message="Поле должно содержать число в 16-ричный формате"
        />
        <q-select
          v-model="resVariableBit"
          :options="allVariables"
          label="Результирующая переменная"
          :rules="[ val => val !== null || 'Обязательное поле' ]"
          bottom-slots
          hint="Обязательное поле" />
        <q-input
          v-model="descriptionBitwise"
          filled
          type="textarea"
          label="Описание"
        />
        <div>
          <q-btn label="Submit" type="submit" color="primary"/>
          <q-btn label="Reset" type="reset" color="primary" flat class="q-ml-sm" />
        </div>
      </q-form>
    </div>
  </q-card>
</template>

<script>
import { uid } from 'quasar';

export default {
  name: 'Form',
  props: {
    action: String,
    allVariables: Array,
    data: Array,
    strVariables: Array,
    editingInd: Number,
    editingIdCreate: String,
    editingObj: Object,
  },
  data() {
    return {
      editing: false,
      name: null,
      type: null,
      types: [
        'boolean', 'bytes', 'double', 'integer', 'long', 'string',
      ],
      size: 0,
      value: '',
      description: null,
      valueSet: null,
      variableSet: null,
      descriptionSet: '',
      variableReplace: null,
      oldStr: '',
      newStr: '',
      resVariable: null,
      descriptionReplace: '',
      variableBit: null,
      typeBit: null,
      typesBit: [
        'AND', 'OR', 'XOR',
      ],
      mask: '',
      resVariableBit: null,
      descriptionBitwise: '',
      tmpActions: [],
    };
  },
  watch: {
    editingObj() {
      this.editing = true;
      switch (this.editingObj.action) {
        case 'CREATE':
          this.editCreate();
          break;
        case 'SET':
          this.editSet();
          break;
        case 'REPLACE':
          this.editReplace();
          break;
        case 'BITWISE':
          this.editBitwise();
          break;
        default:
          break;
      }
    },
    variableSet() {
      if (this.action === 'SET') {
        const variable = this.data.find((item) => item.name === this.variableSet);
        const { value } = variable;
        this.valueSet = value;
      }
    },
  },
  computed: {
    isValidName() {
      return /^[A-Za-z_]*$/.test(this.name);
    },
    isValidMask() {
      return /^[0-9ABCDEF]*$/.test(this.mask);
    },
    isValidValueSet() {
      if ((this.variableSet) && (this.valueSet)) {
        const varInd = this.data.findIndex((item) => item.name === this.variableSet);
        const varSize = this.data[varInd].size;
        const varType = this.data[varInd].type;
        return ((this.valueSet.length <= varSize) && (this.isValidVal(varType, this.valueSet)));
      }
      return true;
    },
  },
  methods: {
    onSubmitCreate() {
      this.newCreateObj = {
        id: uid(),
        action: 'CREATE',
        name: this.name,
        type: this.type,
        size: this.size,
        value: this.value,
        description: this.description,
        actions: [],
      };
      if (this.editing) {
        this.newCreateObj.actions = this.tmpActions;
        this.$emit('replaceCreate', this.editingIdCreate, this.newCreateObj, this.editingObj);
        this.editing = false;
      } else {
        this.$emit('create', this.newCreateObj);
      }
      this.$emit('close');
      this.onReset(this.name, this.type, this.size, this.description);
    },
    onSubmitSet() {
      this.newSetObj = {
        id: uid(),
        action: 'SET',
        variableName: this.variableSet,
        newValue: this.valueSet,
        description: this.descriptionSet,
      };
      if (this.editing) {
        this.$emit('replace', this.editingIdCreate, this.newSetObj, this.editingInd);
        this.editing = false;
      } else {
        this.$emit('add', this.variableSet, this.newSetObj);
      }
      this.$emit('close');
      this.onReset();
    },
    onSubmitReplace() {
      this.newReplaceObj = {
        id: uid(),
        action: 'REPLACE',
        variableName: this.variableReplace,
        oldStr: this.oldStr,
        newStr: this.newStr,
        variableResult: this.resVariable,
        description: this.descriptionReplace,
      };
      if (this.editing) {
        this.$emit('replace', this.editingIdCreate, this.newReplaceObj, this.editingInd);
        this.editing = false;
      } else {
        this.$emit('add', this.variableReplace, this.newReplaceObj);
      }
      this.$emit('close');
      this.onReset();
    },
    onSubmitBitwise() {
      this.newBitwiseObj = {
        id: uid(),
        action: 'BITWISE',
        variableName: this.variableBit,
        operation: this.typeBit,
        mask: this.mask,
        variableResult: this.resVariableBit,
        description: this.descriptionBitwise,
      };
      if (this.editing) {
        this.$emit('replace', this.editingIdCreate, this.newBitwiseObj, this.editingInd);
        this.editing = false;
      } else {
        this.$emit('add', this.variableBit, this.newBitwiseObj);
      }
      this.$emit('close');
      this.onReset();
    },
    onReset() {
      this.name = null;
      this.type = null;
      this.size = 0;
      this.value = '';
      this.description = null;
      this.valueSet = null;
      this.variableSet = null;
      this.descriptionSet = '';
      this.variableReplace = null;
      this.oldStr = '';
      this.newStr = '';
      this.resVariable = null;
      this.descriptionReplace = '';
      this.variableBit = null;
      this.typeBit = null;
      this.mask = '';
      this.resVariableBit = null;
      this.descriptionBitwise = '';
    },
    isValidVal(type, val) {
      if (type === 'boolean') {
        return /^(true|false)$/.test(val) || !val;
      } if (type === 'bytes') {
        return /^[ 0-1]+$/.test(val) || !val;
      } if (type === 'double') {
        return /^\d+(?:[.,]\d+)?$/.test(val) || !val;
      } if ((type === 'integer') || ((type === 'long'))) {
        return /^[-+]?\d*$/.test(val) || !val;
      } if (type === 'string') {
        return true;
      }
      return true;
    },
    isValidValue(type, val, size) {
      return ((this.isValidVal(type, val)) && (val.length <= size));
    },
    editCreate() {
      this.name = this.editingObj.name;
      this.type = this.editingObj.type;
      this.size = this.editingObj.size;
      this.value = this.editingObj.value;
      this.description = this.editingObj.description;
      this.tmpActions = this.editingObj.actions;
    },
    editSet() {
      this.variableSet = this.editingObj.variableName;
      this.valueSet = this.editingObj.newValue;
      this.descriptionSet = this.editingObj.description;
    },
    editReplace() {
      this.variableReplace = this.editingObj.variableName;
      this.oldStr = this.editingObj.oldStr;
      this.newStr = this.editingObj.newStr;
      this.resVariable = this.editingObj.variableResult;
      this.descriptionReplace = this.editingObj.description;
    },
    editBitwise() {
      this.variableBit = this.editingObj.variableName;
      this.typeBit = this.editingObj.operation;
      this.mask = this.editingObj.mask;
      this.resVariableBit = this.editingObj.variableResult;
      this.descriptionBitwise = this.editingObj.description;
    },
  },
};
</script>

<style scoped>

</style>
