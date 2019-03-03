<template>
  <v-container   grid-list-md text-xs-center>
    <v-layout row wrap>
      <v-flex class="headline green whitetext" xs12>
        {{
          blackScoreByPoints > greenScoreByPoints ?
          'GREEN' : 'BLACK'
        }}
        Team has the lead with a {{
          blackScoreByPoints < greenScoreByPoints ?
          greenScoreByPoints-blackScoreByPoints :
          blackScoreByPoints-greenScoreByPoints
        }} point difference <BR></BR>
      </v-flex>
        <v-flex class="black whitetext" xs12>
          (note: the higher your placing on the FCCF leaderboard the lower your points)
        </v-flex>
      <v-flex  class="size whitetext" xs6>
        <div class="green">
          GREEN POINTS {{ greenScoreByPoints }}
          </br>({{ greenCompleted}}/{{greenAtheletes.length * parseInt(week, 10)}} workouts completed and confirmed)
          <ul v-for="(item, key) in greenAtheletes" :key=key>
            <li  > {{ item.competitorName }} ({{item.completed}}/{{week}}) </li>
          </ul>
        </div>
      </v-flex>
      <v-flex class="size whitetext"  xs6>
        <div class="black">
          BLACK POINTS {{ blackScoreByPoints }}
          </br>({{ blackCompleted}}/{{blackAtheletes.length * parseInt(week, 10)}} workouts completed and confirmed)
          <ul v-for="(item, key) in blackAtheletes" :key=key>
            <li  > {{ item.competitorName }} ({{item.completed}}/{{week}}) </li>
          </ul>

        </div>
      </v-flex>
      <v-flex xs2></v-flex>
      <v-flex align-content-center xs12>
        <div>
        <img src="~assets/images/download.jpeg" alt="Smiley face">
        </div>
      </v-flex>
      <v-flex xs2></v-flex>
    </v-layout>
  </v-container>
</template>
<style>
.blacktext {
  color: black;
}
ul {
  list-style-type: none;
}

img {
  width: 100%;
}
.size {
  height:50rem;
}
.whitetext {
  color:white;
}
div{
  height: 100%;
}
  .green {
    background-color: green;
  }
   .black {
    background-color: black;
  }
</style>
<script>
const _ = require('lodash');


const teamBlack = [ 1063056,936605, 1034054, 1668782, 1654875, 1518645, 1626216, 1668672, 708073, 946872, 708022, 1575134, 1700999, 952877, 1253849, 1275746, 371298, 347291, 1693243]
const teamGreen = [ 1697631,1595926,1202565, 1397696,417420, 973378, 397804, 708103, 773883, 706490, 1262251, 1304578, 1276204, 1632490, 1550233, 687298, 1654168, 1355793, 1677273 ];
export default {
  async asyncData({ $axios }) {
    const mens = await $axios.$get('https://games.crossfit.com/competitions/api/v1/competitions/open/2019/leaderboards?affiliate=13624&division=1&scaled=0&page=1')
    const mensscaled = await $axios.$get('https://games.crossfit.com/competitions/api/v1/competitions/open/2019/leaderboards?affiliate=13624&division=1&scaled=1&page=1')
    const womens = await $axios.$get('https://games.crossfit.com/competitions/api/v1/competitions/open/2019/leaderboards?affiliate=13624&division=2&scaled=0&page=1')
    const womensscaled = await $axios.$get('https://games.crossfit.com/competitions/api/v1/competitions/open/2019/leaderboards?affiliate=13624&division=2&scaled=1&page=1')
    //to get dave
    const mensover5559 = await $axios.$get('https://games.crossfit.com/competitions/api/v1/competitions/open/2019/leaderboards?affiliate=13624&division=7&scaled=0&page=1');
    //to get Emma
    const womensunder1415 = await $axios.$get('https://games.crossfit.com/competitions/api/v1/competitions/open/2019/leaderboards?affiliate=13624&division=15&scaled=0&page=1');
    return { mens, mensscaled, womens, womensscaled, mensover5559, womensunder1415 }
  },

  computed: {
      atheletes () {
        let mens = this.divisionMap(this.mens.leaderboardRows);
        let womens = this.divisionMap(this.womens.leaderboardRows);
        let mensover5559 = this.divisionMap(this.mensover5559.leaderboardRows);
        let womensunder1415 = this.divisionMap(this.womensunder1415.leaderboardRows);
        return this.addPoints(womens.concat(mens).concat(mensover5559).concat(womensunder1415));
      },
      week(){
        return this.mens.leaderboardRows[0].scores.length;
      },
      greenAtheletes(){
        return teamGreen.map(val => _.find(this.atheletes, athelete => athelete.competitorId == val)
        );
      },
      blackAtheletes(){
        return teamBlack.map(val => _.find(this.atheletes, athelete => athelete.competitorId == val)
        );
      },
      blackScoreByScore(){
        return teamBlack.reduce((accum, val, key)=>{
          accum += parseInt(_.find(this.atheletes, athelete => athelete.competitorId == val).totalScore,10);
          return accum
        }, 0);
        //return test;
      },
      blackScoreByPoints () {
        return parseInt(teamBlack.reduce((accum, val, key)=>{
          accum += parseInt(_.find(this.atheletes, athelete => athelete.competitorId == val).points,10);
          return accum
        }, 0), 10);
      },
      greenScoreByScore(){
        return parseInt(teamGreen.reduce((accum, val, key)=>{
          accum += parseInt(_.find(this.atheletes, athelete => athelete.competitorId == val).totalScore,10);
          return accum
        }, 0));
      },
      greenScoreByPoints(){
        return parseInt(teamGreen.reduce((accum, val, key)=>{
          accum += parseInt(_.find(this.atheletes, athelete => athelete.competitorId == val).points,10);
          return accum
        }, 0));
      },
      greenCompleted(){
         return parseInt(teamGreen.reduce((accum, val, key)=>{
          accum += parseInt(_.find(this.atheletes, athelete => athelete.competitorId == val).completed);
          return accum
        }, 0));
      },
       blackCompleted(){
         return parseInt(teamBlack.reduce((accum, val, key)=>{
          accum += parseInt(_.find(this.atheletes, athelete => athelete.competitorId == val).completed);
          return accum
        }, 0));
      }
    },

    methods: {
      divisionMap(data) {
        if (data) {
        return data.map(athlete => {
          return {...athlete.entrant, scores: [...athlete.scores], overallRank: athlete.overallRank }
        });
        } else {
          return []
        }
      },
      addTotalScore(atheletes){
       return atheletes.map(athelete =>{
          let score = athelete.scores.reduce((accum, score, key) => {
            accum += parseInt(score.score)
            return accum;
          }, 0);
          let completed = athelete.scores.filter(score => score.score !== '0').length;
          return { ...athelete, totalScore:score, completed:completed };
        });
      },
      sortAtheltesByScore(atheletes){
        return atheletes.sort((a,b) => {
          if ( a.totalScore < b.totalScore){
            return 1;
          }
          if ( a.totalScore > b.totalScore){
            return -1;
          }
          return 0;
        });
      },
      calculatePoints(totals){
        var points = 2;
        let numTie = 0;
        for(let x=0; x < totals.length; x++){
          if(totals[x] && totals[x+1]){
            if (totals[x].totalScore === totals[x+1].totalScore){
              numTie++;
              totals[x].points = points;
            } else {
              totals[x].points = points;
              points = points+numTie;
              numTie = 0;
              points++;
            }
          } else {
            totals[x].points = points;
          }
        }
        return totals;
      },
      addPoints(atheletes){
        let totals = this.addTotalScore(atheletes);
        totals = this.sortAtheltesByScore(totals);
        totals = this.calculatePoints(totals);

        return totals;
      },
    }
}
</script>
