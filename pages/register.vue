<template>
  <v-container>
    <validation-observer
      ref="observer"
      v-slot="{ invalid }"
    >
      <form @submit.prevent="submit">
        <v-row justify="center">
          <v-col
            cols="12"
            md="7"
            class="py-1"
          >
            <validation-provider
              v-slot="{ errors }"
              name="名前"
              rules="required"
            >
              <v-text-field
                v-model="inputPlayerName"
                :error-messages="errors"
                label="名前"
                required
              ></v-text-field>
            </validation-provider>
          </v-col>

          <v-col
            cols="12"
            md="7"
            class="py-1"
          >
            <validation-provider
              v-slot="{ errors }"
              name="技能証"
              rules="required"
            >
              <v-select
                v-model="selectedSkill"
                :error-messages="errors"
                :items="skills"
                label="技能証"
                required
              ></v-select>
            </validation-provider>
          </v-col>

          <v-col
            cols="12"
            md="7"
            class="py-1"
          >
            <validation-provider
              v-slot="{ errors }"
              name="グライダー名"
              rules="required"
            >
              <v-text-field
                v-model="inputGliderName"
                :error-messages="errors"
                label="グライダー名"
                required
              ></v-text-field>
            </validation-provider>
          </v-col>
    
          <v-col
            cols="12"
            md="7"
            class="py-1"
          >
            <validation-provider
              v-slot="{ errors }"
              name="グライダータイプ"
              rules="required"
            >
              <v-select
                v-model="selectedGliderType"
                :error-messages="errors"
                :items="gliderTypes"
                label="グライダータイプ"
                required
              ></v-select>
            </validation-provider>
          </v-col>

          <v-col
            cols="12"
            md="7"
            class="py-1"
          >
            <v-dialog
              v-model="dialog"
              persistent
              max-width="290"
            >
              <template v-slot:activator="{ on, attrs }">
                <v-row>
                  <v-btn
                    color="success"
                    class="mx-auto mt-8"
                    dark
                    :disabled="invalid"
                    v-bind="attrs"
                    v-on="on"
                  >
                    登録する
                  </v-btn>
                </v-row>
              </template>
              <v-card>
                <v-card-title class="text-h5">
                  選手を登録します
                </v-card-title>
                <v-card-text>名前：{{ inputPlayerName }}</v-card-text>
                <v-card-text>技能証：{{ selectedSkill }}</v-card-text>
                <v-card-text>グライダー名：{{ inputGliderName }}</v-card-text>
                <v-card-text>グライダータイプ：{{ selectedGliderType }}</v-card-text>
                <v-card-actions>
                  <v-btn
                    color="error"
                    class="mx-2 mb-4"
                    @click="dialog = false"
                  >
                    戻る
                  </v-btn>
                  <v-spacer></v-spacer>
                  <v-btn
                    color="success"
                    class="mx-2 mb-4"
                    @click="registerPlayer"
                  >
                    OK
                  </v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>
          </v-col>
        </v-row>
      </form>
    </validation-observer>
  </v-container>
</template>

<script>
import { required } from "vee-validate/dist/rules"
  import { extend, ValidationObserver, ValidationProvider } from "vee-validate"

  extend("required", {
    ...required,
    message: "{_field_} が未記入です",
  })

export default {
  components: {
    ValidationProvider,
    ValidationObserver,
  },
  data: () => ({
    header: {
      title: "選手登録"
    },
    valid: true,
    dialog: false,
    
    inputPlayerName: "",
    selectedSkill: "",
    skills: [
      "C",
      "P",
      "XCP",
    ],
    inputGliderName: "",
    selectedGliderType: "",
    gliderTypes: [
      "ツノあり",
      "ツノなし",
    ],
  }),
  mounted() {
    this.updateHeader();
  },
  methods: {
    updateHeader() {
      this.$nuxt.$emit("updateHeader", this.header.title );
    },

    async registerPlayer() {
      try {
        const result = await this.$axios.post("/register",
          {
            player_name: this.inputPlayerName,
            has_xcp: (this.selectedSkill == "XCP"),
            glider_name: this.inputGliderName,
            has_kingpost: (this.selectedGliderType == "ツノなし"),
          }
        );
        this.resetForm();
      } catch(error) {
        this.dialog = false;
        alert("登録できませんでした");
      }
    },

    resetForm() {
      this.dialog = false;
      this.inputPlayerName = "";
      this.selectedSkill = "";
      this.inputGliderName = "";
      this.selectedGliderType = "";
      this.$refs.observer.reset();
    },
  }
}
</script>