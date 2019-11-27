<template>
  <div class="addInput" :id="mainId">
    <div
      :class="['inputRow', ...mainClass]"
      v-for="lineNumber in linesNumber" :key="lineNumber"
    >
      <div v-for="(field, index) in fields" :key="index">
        <v-text-field
          v-if="field.mask"
          :class="field.class"
          :label="field.label"
          :hint="getHint(lineNumber, field.hint)"
          persistent-hint
          v-model="fieldData[field.dataName][lineNumber -1]"
          @input="dataChange()"
          v-mask="field.mask"
          @blur="touch(field.dataName, lineNumber -1)"
          :error-messages="errors(field.dataName, lineNumber -1)"
        >
          <template v-if="index === fields.length -1" slot="append">
            <v-icon
              v-if="linesNumber > 1"
              @click="fieldMinus(lineNumber -1), dataChange()"
            >mdi-minus-circle-outline</v-icon>
            <div v-if="lineNumber === linesNumber">
              <v-tooltip bottom v-if="tooltip">
                <template v-slot:activator="{ on }">
                  <v-icon @click="fieldPlus" v-on="on">mdi-plus-circle-outline</v-icon>
                </template>
                <span>{{ tooltip }}</span>
              </v-tooltip>
              <v-icon v-else @click="fieldPlus">mdi-plus-circle-outline</v-icon>
            </div>
          </template>
        </v-text-field>

        <v-text-field
          v-else
          :class="field.class"
          :label="field.label"
          :hint="getHint(lineNumber, field.hint)"
          persistent-hint
          v-model="fieldData[field.dataName][lineNumber -1]"
          @input="dataChange()"
          @blur="touch(field.dataName, lineNumber -1)"
          :error-messages="errors(field.dataName, lineNumber -1)"
        >
          <template v-if="index === fields.length -1" slot="append">
            <v-icon
              v-if="linesNumber > 1"
              @click="fieldMinus(lineNumber -1), dataChange()"
            >mdi-minus-circle-outline</v-icon>
            <div v-if="lineNumber === linesNumber">
              <v-tooltip bottom v-if="tooltip">
                <template v-slot:activator="{ on }">
                  <v-icon @click="fieldPlus" v-on="on">mdi-plus-circle-outline</v-icon>
                </template>
                <span>{{ tooltip }}</span>
              </v-tooltip>
              <v-icon v-else @click="fieldPlus">mdi-plus-circle-outline</v-icon>
            </div>
          </template>
        </v-text-field>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'addInput',
  props: {
    tooltip: {
      // Tooltip on the + button
      type: String,
      required: false,
      default: null,
    },
    mainClass: {
      type: Array[String],
      required: false,
      default: null,
    },
    mainId: {
      type: String,
      required: false,
      default: null,
    },
    fields: {
      // All the text-input you need
      // Shape of object: {dataName, label, class, hint, mask, validations: [{name, rule, message}]}
      type: Array[Object],
      required: true,
    },
  },
  data() {
    return {
      linesNumber: 1,
      validations: {},
      fieldData: {},
    };
  },
  created() {
    // Links the text-input to their data according to the props 'fields'
    // FieldData is an object of array with fields.dataName like object key
    this.fieldData = this.fields.reduce((acc, field) => ({ ...acc, [field.dataName]: [] }), {});

    // Links the dataName to their validations
    // Validations is an object of array with fields.dataName like object key
    this.validations = this.fields.reduce((acc, field) => ({
      ...acc, [field.dataName]: field.validations || [],
    }), {});
  },
  methods: {
    getHint(lineNumber, hint) {
      return lineNumber === this.linesNumber ? hint : null;
    },
    fieldPlus() {
      this.linesNumber += 1;
    },
    fieldMinus(index) {
      this.linesNumber -= 1;
      // Delete the great data in each array of fieldData
      Object.keys(this.fieldData).forEach((key) => {
        this.fieldData[key].splice(index, 1);
      });
    },
    errors(name, index) {
      const input = this.$v.fieldData[name].$each.$iter[index];
      // Display errors of an text-input according to his validation
      if (input && input.$dirty) {
        return this.validations[name].reduce((acc, validation) => {
          if (!input[validation.name]) {
            return [...acc, validation.message];
          }
          return acc;
        }, []);
      }
      return null;
    },
    touch(name, index) {
      // Assign the validation of an text-input to 'dirty'
      const input = this.$v.fieldData[name].$each.$iter[index];
      if (input) {
        input.$touch();
      }
    },
    dataChange() {
      // Emit the all the data
      // For each key in fieldData object
      const data = Object.keys(this.fieldData).reduce((acc1, key) => {
        // If one field is invalid assign attribute to []
        if (this.$v.fieldData[key].$invalid) {
          return { ...acc1, [key]: [] };
        }
        // Else fill the attribute with the data
        return {
          ...acc1,
          [key]: this.fieldData[key].reduce((acc2, field, index) => {
            // Not assign this field if he is empty
            if (field && field !== '' && !this.$v.fieldData[key].$each.$iter[index].$invalid) {
              return [...acc2, field];
            }
            return acc2;
          }, []),
        };
      }, {});
      this.$emit('addInputChange', data);
    },
  },
  // vuelidate method
  validations() {
    const v = {
      fieldData: {},
    };
    this.fields.forEach((field) => {
      const validations = (field.validations || []).reduce((acc, validation) => {
        const data = { ...acc, [validation.name]: validation.rule };
        return data;
      }, {});
      v.fieldData[field.dataName] = {
        $each: validations,
      };
    });
    return v;
  },
};
</script>

<style lang="scss" scoped>
@import "@/assets/scss/base/_variables.scss";

.addInput::v-deep {

  > div:first-child label {
    display: initial;
  }
  label {
    display: none;

    @media (max-width: $screen-md-max) {
      display: initial;
    }
  }
  .v-icon {
    color: rgba(40, 40, 40, 0.6);
    height: 18px;
    width: 18px;
    margin: 0 4px;
  }
}
</style>
