<template>
  <v-container>
    <v-row 
      v-for="player in players"
      :key="player.id"
    >
      <v-col 
        cols="1" 
        class="pb-1" 
        style="text-align:right"
      >
        {{ player.rank }}
      </v-col>
      <v-col 
        cols="4" 
        class="pb-1"
      >
        {{ player.name }}
      </v-col>
      <v-col 
        cols="3" 
        class="pb-1"
      >
        {{ player.total_point }}点
      </v-col>
      <v-spacer></v-spacer>
      <v-col 
        cols="1" 
        class="mr-5 pb-1"
        style="text-align:right"
      >
        <v-btn
          v-if="edit"
          color="gray"
          class="p-0 error--text"
          icon
          dark
          small
          @click="openDeletePlayerDialog(player)"
        >
          ✖
        </v-btn>
      </v-col>

      <v-col cols="12" class="mt-0 mb-2 pt-0">
        <v-simple-table 
          dark 
          dense
        >
          <template v-slot:default>
            <thead>
              <tr>
                <th 
                  class="text-center"
                  style="width:100px;"
                >
                  <!-- 本数 -->
                </th>
                <th class="text-center">
                  時
                </th>
                <th class="text-center">
                  LD
                </th>
                <th class="text-center">
                  タ
                </th>
                <th class="text-center">
                  パ
                </th>
                <th 
                  v-if="edit" 
                  class="text-center"
                  style="width:20px;"
                >
                  <!-- 削除ボタン -->
                </th>
              </tr>
            </thead>
            <tbody>
              <tr
                v-for="(flight, index) in selectedPlayerFlights(player.id)"
                :key="flight.id"
              >
                <td 
                  class="text-center px-0 caption" 
                >
                  {{ index+1 }}本目
                </td>
                <td class="text-center px-0">
                  {{ flight.time_point }}
                </td>
                <td class="text-center px-0">
                  {{ flight.landing_point }}
                </td>
                <td class="text-center px-0">
                  {{ flight.target_point }}
                </td>
                <td class="text-center px-0">
                  {{ pylonPoint(flight) }}
                </td>
                <td 
                  v-if="edit"
                  class="text-center px-0" 
                >
                  <v-btn
                    color="gray"
                    class="p-0 error--text"
                    icon
                    dark
                    small
                    @click="openDeleteFlightDialog(flight, player)"
                  >
                    ✖
                  </v-btn>
                </td>
              </tr>
            </tbody>
          </template>
        </v-simple-table>
      </v-col>
    </v-row>

    <v-dialog
      v-model="dialogP"
      persistent
      max-width="290"
    >
      <v-card>
        <v-card-title class="text-h5 red--text">
          選手を削除します
        </v-card-title>
        <v-card-text>{{ clickedPlayer.name }}さん</v-card-text>
        <v-card-text>・技能証：{{ clickedPlayer.has_xcp == true ? "XCP" : "C/P" }}</v-card-text>
        <v-card-text>・グライダー名：{{ clickedPlayer.glider_name }}</v-card-text>
        <v-card-text>・グライダータイプ：{{ clickedPlayer.has_kingpost == true ? "ツノなし" : "ツノあり" }}</v-card-text>
        <v-card-actions>
          <v-btn
            color="success"
            class="mx-2 mb-4"
            @click="dialogP = false"
          >
            戻る
          </v-btn>
          <v-spacer></v-spacer>
          <v-btn
            color="error"
            class="mx-2 mb-4"
            @click="deletePlayer(clickedPlayer.id)"
          >
            OK
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog
      v-model="dialogF"
      persistent
      max-width="290"
    >
      <v-card>
        <v-card-title class="text-h5 red--text">
          記録を削除します
        </v-card-title>
        <v-card-text>{{ clickedFlight.player_name }}さん</v-card-text>
        <v-card-text>・時間得点：{{ clickedFlight.time_point }}点</v-card-text>
        <v-card-text>・LD得点：{{ clickedFlight.landing_point }}点</v-card-text>
        <v-card-text>・ターゲット得点：{{ clickedFlight.target_point }}点</v-card-text>
        <v-card-text>・パイロン得点：{{ clickedFlight.pylon_point }}点</v-card-text>
        <v-card-actions>
          <v-btn
            color="success"
            class="mx-2 mb-4"
            @click="dialogF = false"
          >
            戻る
          </v-btn>
          <v-spacer></v-spacer>
          <v-btn
            color="error"
            class="mx-2 mb-4"
            @click="deleteFlight(clickedFlight.id, clickedFlight.player_id)"
          >
            OK
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
export default {
  data: () => ({
    header: {
      title: "結果表"
    },
    edit: false,
    dialogF: false,
    dialogP: false,
    players: [],
    flights: [],
    clickedPlayer: {},
    clickedFlight: {},
  }),
  created() {
    this.getResults();
    if(this.$route.query.edit != null){
      this.edit = true;
      this.header.title = "結果編集"
    }
  },
  mounted() {
    this.updateHeader();
  },
  computed: {
    selectedPlayerFlights: function() {
      return (
        (playerId) => {
          return this.flights.filter(element => element.player_id == playerId);
        }
      )
    },

    rankedPlayers: function() {
      return (
        (players) => {
          players.forEach( element => {
            element["total_point"] = element.max_time_point
              + element.max_landing_point
              + element.max_target_point
              + element.max_pylon1_point
              + element.max_pylon2_point;
          });

          players.sort( function(a,b) {
              if(a.total_point < b.total_point) return 1;
              if(a.total_point > b.total_point) return -1;
              return 0;
          });

          let point = 100;
          let num = 1;
          let rank = 1;
          players.forEach( element => {
            if(element.total_point != point) {
              rank = num;
            }
            element["rank"] = rank;
            point = element.total_point;
            num++;
          });

          return players;
        }
      )
    },

    pylonPoint: function() {
      return (
        (flight) => {
          return flight.pylon1_point + flight.pylon2_point;
        }
      )
    },
  },
  methods: {
    updateHeader() {
      this.$nuxt.$emit("updateHeader", this.header.title );
    },

    async getResults() {
      const resultsData = await this.$axios.get("/results");
      this.players = this.rankedPlayers(resultsData.data.data.players);
      this.flights = resultsData.data.data.flights;
    },

    openDeletePlayerDialog(player) {
      this.clickedPlayer = player;
      this.dialogP = true;
    },

    openDeleteFlightDialog(flight, player) {
      this.clickedFlight = flight;
      this.clickedFlight.pylon_point = this.pylonPoint(flight);
      this.clickedFlight.player_name = player.name;
      this.dialogF = true;
    },

    async deleteFlight(flightId, playerId) {
      try{
        await this.$axios.post("/results/flight",
          {
            flight_id: flightId,
            player_id: playerId,
          } 
        );
        this.dialogF = false;
        this.getResults();
      } catch(error) {
        this.dialogF = false;
        alert("削除できませんでした");
      }
    },

    async deletePlayer(playerId) {
      try{
        await this.$axios.post("/results/player",
          {
            player_id: playerId,
          } 
        );
        this.dialogP = false;
        this.getResults();
      } catch(error) {
        this.dialogP = false;
        alert("削除できませんでした");
      }
    },
  }
}
</script>