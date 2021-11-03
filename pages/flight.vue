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
              <v-select
                v-model="selectedPlayerId"
                :error-messages="errors"
                :items="players"
                item-text="name"
                item-value="id"
                label="名前"
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
              name="フライト時間（分）"
              rules="required|numeric"
            >
              <v-text-field
                v-model="inputTime"
                :error-messages="errors"
                label="フライト時間（分）"
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
              name="LD得点（点）"
              rules="required|numeric"
            >
              <v-text-field
                v-model="inputLandingPoint"
                :error-messages="errors"
                label="LD得点（点）"
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
              name="ターゲット"
              rules="required"
            >
              <v-select
                v-model="selectedTarget"
                :error-messages="errors"
                :items="target"
                item-text="text"
                item-value="value"
                label="ターゲット"
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
              name="シークレットパイロン（文字入力）"
            >
              <v-text-field
                v-model="inputPylon"
                :error-messages="errors"
                label="シークレットパイロン（文字入力）"
              ></v-text-field>
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
                    記録する
                  </v-btn>
                </v-row>
              </template>
              <v-card>
                <v-card-title class="text-h5">
                  記録を送信します
                </v-card-title>
                <v-card-text>名前：{{ selectedPlayerId }}</v-card-text>
                <v-card-text>フライトタイム：{{ inputTime }}分</v-card-text>
                <v-card-text>LD得点：{{ inputLandingPoint }}点</v-card-text>
                <v-card-text>ターゲット：{{ selectedTarget }}</v-card-text>
                <v-card-text>シークレットパイロン：{{ inputPylon }}</v-card-text>
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
                    @click="sendFlight"
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
import { required, numeric } from "vee-validate/dist/rules"
  import { extend, ValidationObserver, ValidationProvider } from "vee-validate"

  extend("required", {
    ...required,
    message: "{_field_} が未記入です",
  })
  extend("numeric", {
    ...numeric,
    message: "{_field_} は半角整数で入力してください",
  })

export default {
  components: {
    ValidationProvider,
    ValidationObserver,
  },
  data: () => ({
    header: {
      title: "フライト記録"
    },
    valid: true,
    dialog: false,
    
    players: [],
    selectedPlayerId: "",
    inputTime: "",
    inputLandingPoint: "",
    selectedTarget: "",
    target: [
      {text: "イン", value: true },
      {text: "アウト", value: false },
    ],
    inputPylon: "",
  }),
  created() {
    this.getPayerNames();
  },
  mounted() {
    this.updateHeader();
  },
  methods: {
    async getPayerNames() {
      const playersData = await this.$axios.get("/flights");
      this.players = playersData.data.data;
    },

    updateHeader() {
      this.$nuxt.$emit("updateHeader", this.header.title );
    },
    async sendFlight() {
      try {
        const result = await this.$axios.post("/flights",
          {
            player_id: this.selectedPlayerId,
            time: this.inputTime,
            landing: this.inputLandingPoint,
            target: this.selectedTarget,
            pylon: this.inputPylon,
          }
        )
        this.resetForm();
      } catch(error) {
        this.dialog = false;
        alert("記録できませんでした");
      }
    },

    resetForm() {
      this.dialog = false;
      this.selectedPlayerId = "";
      this.inputTime = "";
      this.inputLandingPoint = "";
      this.selectedTarget = "";
      this.inputPylon = "";
      this.$refs.observer.reset();
    },
  }
}
</script>