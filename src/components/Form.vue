<template>
  <q-card class="my-card" v-if="action">
    <div class="q-pa-md">
      <div class="text-h6">{{action}}</div>
      <q-form v-if="action === 'CREATE'"
        @submit="onSubmitCreate"
        @reset="onReset"
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
          hint="Обязательное поле" />
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
          hint="Обязательное поле" />
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
          hint="Обязательное поле" />
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
          hint="Обязательное поле" />
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
    editing: Boolean,
    editingArr: Array,
  },
  data() {
    return {
      name: '',
      type: null,
      types: [
        'boolean', 'bytes', 'double', 'integer', 'long', 'string',
      ],
      size: 0,
      value: '',
      description: '',
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
    };
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
      this.$emit('create', this.newCreateObj);
    },
    onSubmitSet() {
      this.newSetObj = {
        action: 'SET',
        variableName: this.variableSet,
        newValue: this.valueSet,
        description: this.descriptionSet,
      };
      this.$emit('add', this.variableSet, this.newSetObj);
    },
    onSubmitReplace() {
      this.newReplaceObj = {
        action: 'REPLACE',
        variableName: this.variableReplace,
        oldStr: this.oldStr,
        newStr: this.newStr,
        variableResult: this.resVariable,
        description: this.descriptionReplace,
      };
      this.$emit('add', this.variableReplace, this.newReplaceObj);
    },
    onSubmitBitwise() {
      this.newBitwiseObj = {
        action: 'BITWISE',
        variableName: this.variableBit,
        operation: this.typeBit,
        mask: this.mask,
        variableResult: this.resVariableBit,
        description: this.descriptionBitwise,
      };
      this.$emit('add', this.variableBit, this.newBitwiseObj);
    },
    // to do
    onReset() {
    },
    isValidVal(type, val) {
      if (type === 'boolean') {
        return /^(true|false)$/.test(val) || !val;
      } if (type === 'bytes') {
        return /^[ 0-1]+$/.test(val) || !val;
      } if (type === 'double') {
        return /^\d+(?:[.,]\d+)?$/.test(val) || !val;
      } if ((type === 'integer') || ((val === 'long'))) {
        return /^[-+]?\d*$/.test(val) || !val;
      } if (type === 'string') {
        return true;
      }
      return true;
    },
    isValidValue(type, val, size) {
      return ((this.isValidVal(type, val)) && (val.length <= size));
    },
  },
};
</script>

<style scoped>

</style>
