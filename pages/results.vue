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
      <v-spacer></v-spacer>
      <v-col 
        cols="2" 
        class="pb-1"
      >
        {{ player.total_point }}点
      </v-col>

      <v-col cols="12" class="mt-0 mb-2 pt-0">
        <v-simple-table 
          dark 
          dense
        >
          <template v-slot:default>
            <thead>
              <tr>
                <th class="text-center">
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
                <th class="text-center">
                  <!-- 削除ボタン -->
                </th>
              </tr>
            </thead>
            <tbody>
              <tr
                v-for="(flight, index) in selectedPlayersFlights(player.id)"
                :key="flight.id"
              >
                <td class="text-center">
                  {{ index+1 }}本目
                </td>
                <td class="text-center">
                  {{ flight.time_point }}
                </td>
                <td class="text-center">
                  {{ flight.landing_point }}
                </td>
                <td class="text-center">
                  {{ flight.target_point }}
                </td>
                <td class="text-center">
                  {{ pylonPoint(flight) }}
                </td>
                <td>
                  <v-btn
                    color="gray"
                    class="p-0 error--text"
                    icon
                    dark
                    small
                    @click="setClickedFlight(flight, player)"
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
      v-model="dialog"
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
            @click="dialog = false"
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
    dialog: false,
    players: [],
    flights: [],
    clickedFlight: {},
  }),
  created() {
    this.getResults();
  },
  mounted() {
    this.updateHeader();
  },
  computed: {
    selectedPlayersFlights: function() {
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

    setClickedFlight(flight, player) {
      this.clickedFlight = flight;
      this.clickedFlight.pylon_point = this.pylonPoint(flight);
      this.clickedFlight.player_name = player.name;
      this.dialog = true;
    },

    async deleteFlight(flightId, playerId) {
      try{
        const results = await this.$axios.post("/results",
          {
            flight_id: flightId,
            player_id: playerId,
          } 
        );
        this.dialog = false;
        this.getResults();
      } catch(error) {
        this.dialog = false;
        alert("削除できませんでした");
      }
    },
  }
}
</script>