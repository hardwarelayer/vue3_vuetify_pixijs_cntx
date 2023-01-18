<template>

  <div class="text-center">
    <br/>DlgForm Show: {{ showForm }}
    <v-dialog
      transition="dialog-top-transition"
      v-model="showForm"
      width="500"
    >
      <v-form ref="form">
      <v-card>

        <v-card-title>
          <span class="text-h5">Input test</span>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-row>
              <v-col cols="12"
              >
                <v-text-field
                  id="first_name"
                  label="Legal first name*"
                  :value="first_name"
                  :rules="[v => !!v || 'Name is required!!!', v => (v && v.length <= 10) || 'Name must be less than 10 characters']"
                  @input="$emit('update:first_name', $event.target.value)"
                  required
                ></v-text-field>
              </v-col>
              <v-col cols="12">
                <v-text-field
                  label="Email*"
                  required
                ></v-text-field>
              </v-col>
              <v-col
                cols="12"
                sm="6"
              >
                <v-select
                  id="age"
                  :items="['0-17', '18-29', '30-54', '54+']"
                  label="Age*"
                  :value="age"
                  @input="$emit('update:age', $event.target.value)"
                  required
                ></v-select>
              </v-col>
            </v-row>
          </v-container>
          <small>*indicates required field</small>
        </v-card-text>

        <v-divider></v-divider>

        <v-card-actions>

          <v-spacer></v-spacer>
          <v-btn
            color="primary"
            text
            @click="$emit('closeVMOFnc')"
          >
            I accept
          </v-btn>
          <v-btn
            color="blue-darken-1"
            variant="text"
            @click="clickSaveFnc($emit, this)"
          >
            Save
          </v-btn>

        </v-card-actions>
      </v-card>
    </v-form>
    </v-dialog>
  </div>

</template>

<script>
  export default {
    methods: {
      clickSaveFnc: function($emit, pt) {
        pt.$refs.form.validate();
        if (first_name.value.length > 0)
            $emit('saveDialogFormFnc');
      },
    },
    props: ["visible", "first_name", "age"],
    nameRules: [
      v => !!v || 'Name is required',
      v => (v && v.length <= 10) || 'Name must be less than 10 characters',
    ],
    computed : {
      showForm: {
        get() {
          return this.visible
        },
        set(value) {
          if (!value) {
            this.$emit("saveDialogFormFnc")
          }
        }
      }
    }
  }

</script>