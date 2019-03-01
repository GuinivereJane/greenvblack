<template>
  <v-container  grid-list-md text-xs-center>
    <v-layout row wrap>
      <v-flex  xs6>
        <v-card dark color="green" height="50rem">
          <v-card-text class="px-0">
            GREEN SCORE {{ greenScoreByScore }}</br>
            GREEN POINTS {{ greenScoreByPoints }}
          </v-card-text>
        </v-card>
      </v-flex>
      <v-flex  xs6>
       <v-card dark height="50rem">
          <v-card-text color="green" class="px-0">
           BLACK SCORE {{ blackScoreByScore }}<br>
           BLACK POINTS {{ blackScoreByPoints }}
          </v-card-text>
        </v-card>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
const _ = require('lodash');
//balck
//Green Dave, Emma

const teamBlack = [1668782, 1654875, 1518645, 1626216, 1668672, 708073, 946872, 708022, 1575134, 1700999, 952877, 1253849, 1275746, 371298, 347291, 1693243]
const teamGreen = [ 417420, 973378, 397804, 708103, 773883, 706490, 1262251, 1304578, 1276204, 1632490, 1550233, 687298, 1654168, 1355793, 1677273 ];
export default {
  async asyncData({ $axios }) {
    const mens = await $axios.$get('https://games.crossfit.com/competitions/api/v1/competitions/open/2019/leaderboards?affiliate=13624&division=1&scaled=0&page=1')
    const mensscaled = await $axios.$get('https://games.crossfit.com/competitions/api/v1/competitions/open/2019/leaderboards?affiliate=13624&division=1&scaled=1&page=1')
    const womens = await $axios.$get('https://games.crossfit.com/competitions/api/v1/competitions/open/2019/leaderboards?affiliate=13624&division=2&scaled=0&page=1')
    const womensscaled = await $axios.$get('https://games.crossfit.com/competitions/api/v1/competitions/open/2019/leaderboards?affiliate=13624&division=2&scaled=1&page=1')
    return { mens, mensscaled, womens, womensscaled }
  },
  computed: {
      atheletes () {
        let mens = this.divisionMap(this.mens.leaderboardRows);
        let womens = this.divisionMap(this.womens.leaderboardRows);
        return womens.concat(mens);
      },
      blackScoreByScore(){
        let test = teamBlack.reduce((accum, val, key)=>{
          accum += parseInt(_.find(this.atheletes, athelete => athelete.competitorId == val).scores[0].score,10);
          return accum
        }, 0);
        return test;
      },
      blackScoreByPoints () {
        console.log(teamBlack.length);
        console.log('----------------');
        let test = teamBlack.reduce((accum, val, key)=>{
          console.log(parseInt(_.find(this.atheletes, athelete => athelete.competitorId == val).overallScore,10));
          accum += parseInt(_.find(this.atheletes, athelete => athelete.competitorId == val).overallScore,10);
          return accum
        }, 0);
        return test;
      },
      greenScoreByScore(){
        let test = teamGreen.reduce((accum, val, key)=>{
          accum += parseInt(_.find(this.atheletes, athelete => athelete.competitorId == val).scores[0].score,10);
          return accum
        }, 0);
        return test;
      },
      greenScoreByPoints(){
        let test = teamGreen.reduce((accum, val, key)=>{
          accum += parseInt(_.find(this.atheletes, athelete => athelete.competitorId == val).overallScore,10);
          return accum
        }, 0);
        return test;
      },


    },

    methods: {
      divisionMap(data) {
        if (data) {
        return data.map(athlete => {
          return {...athlete.entrant, scores: [...athlete.scores], overallRank: athlete.overallRank ,overallScore: athlete.overallScore}
        });
        } else {
          return []
        }
      },

      getScoreFromLeaderboardData (leaderboardRows) {

      }
    }
}
</script>
