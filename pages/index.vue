<template>
  <v-container   grid-list-md text-xs-center>
    <v-layout row wrap>
      <v-flex class="headline green whitetext" xs12>
        {{
          overallScoresWeighted[0] > overallScoresWeighted[1] ?
          'GREEN' : 'BLACK'
        }}
        Team has the lead with a {{
          overallScoresWeighted[0] < overallScoresWeighted[1] ?
          overallScoresWeighted[1]-overallScoresWeighted[0] :
          overallScoresWeighted[0]-overallScoresWeighted[1]
        }} point difference <BR></BR>
      </v-flex>


         <v-flex xs12>
        <div class="resultContainer">
          <v-layout class="black" align-center justify-space-around row fill-height>
             <div class=" black whitetext" v-for="(scores, key) in perRoundWeighted" :key=key+scores >
              <div>ROUND #{{ key + 1}}</div>
              <div>
                {{scores[0] > scores[1] ?'GREEN' : 'BLACK'}}
                BY
                {{
                  scores[0] < scores[1] ?
                  scores[1]-scores[0] :
                  scores[0]-scores[1]
                }}


              </div>
        </div>
          </v-layout>
        </div>
        </v-flex>




        <v-flex class="green whitetext" xs12>
          (note: the higher your placing on the FCCF leaderboard the lower your points)
        </v-flex>
      <v-flex  class="size whitetext" xs6>
        <div class="green">
          GREEN POINTS {{ overallScoresWeighted[1] }}
          </br>({{ greenCompleted}}/{{greenAtheletes.length * parseInt(week, 10)}} workouts completed and confirmed)
          <ul v-for="(item, key) in greenAtheletes" :key=key>
            <li  > {{ item.competitorName }} ({{item.completed}}/{{week}}) </li>
          </ul>
        </div>
      </v-flex>
      <v-flex class="size whitetext"  xs6>
        <div class="black">
          BLACK POINTS {{ overallScoresWeighted[0] }}
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
ul{
  padding-left:0px;
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
      nullsPerRound(){
         const greenNulls = this.countNull(teamGreen);
         const blackNulls = this.countNull(teamBlack);
         return { greenNulls, blackNulls };


          //pointsPerRound[] has been added to each athelete, athelete gets null if the did not complete
          //need to count the number of nulls in each group
          //sort each group
          //remove the nulls,
          //the remove  highest(worst)scoring atheletes from the team with the most competitors until the largest team is the same size as the smallest team
          //return and object {blackScores:[r1,r2,r3...], greenScores[r1,r2,r3...]}

      },
      perRoundWeighted(){
        //returns [back score, green score ]
        let scores = [];
        for (let round = 0; round < this.week; round++){
          let sorted = this.sortAtheltesByScoreByRound(this.atheletes, round);
          let teamBlackRound = sorted.filter(athelete => teamBlack.includes(parseInt(athelete.competitorId)));
          let teamGreenRound = sorted.filter(athelete => teamGreen.includes(parseInt(athelete.competitorId)));
          scores.push([ Math.round(this.wieghtedScoresByRound(round, teamBlackRound)), Math.round(this.wieghtedScoresByRound(round, teamGreenRound)) ]);
        }
        return scores;
      },


      overallScoresWeighted(){
        let scores = this.perRoundWeighted.reduce((accum, val, key) =>{
          accum[0] += val[0];
          accum[1] += val[1];
          return accum;
        }, [0,0]);
        scores = [ scores[0]/this.week, scores[1]/this.week]
        return scores;
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

       scoreByRound(round, trimedTeam){
        return trimedTeam.reduce((accum, athelete, key)=>{
          if (athelete.pointsPerRound[round] !== null){
            accum += athelete.pointsPerRound[round];
          }
          return accum
        }, 0);
      },


      wieghtedScoresByRound(round, team){
        let score = this.scoreByRound(round, team)
        let trimedLength = team.filter(athelete => athelete.pointsPerRound[round] !== null).length;
        let average = score / trimedLength;

        team.forEach(athelete => {
          if (athelete.pointsPerRound[round] === null){
            let total = athelete.pointsPerRound.reduce((accum, val, key) =>{
              if (val !== null){
                accum += val;
              }
              return accum;
            },0);
            let weightedScore = (total + average) / athelete.pointsPerRound.length;
            score += weightedScore;
          }
        });
        return score;
      },
      countNull(team){
        let nulls = [];
          for (let round = 0; round < this.week; round++){
            let count = 0;
            count = team.reduce((accum, val, key) => {
              let score = _.find(this.atheletes, athelete => athelete.competitorId == val).pointsPerRound[round];
              if ( score === null){
                accum++;
              }
              return accum;
            }, 0);
            nulls.push(count);
          }
          return nulls
      },
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
      sortAtheltesByScoreByRound(atheletes, round){
        return atheletes.sort((a,b) => {
          if ( parseInt(a.scores[round].score) < parseInt(b.scores[round].score)){
            return 1;
          }
          if ( parseInt(a.scores[round].score) > parseInt(b.scores[round].score)){
            return -1;
          }
          return 0;
        });
      },
      calculatePointsByRound(totals, round){
        //send in a sorted list of atheletes and the round number
        var points = 2;
        let numTie = 0;
        for(let x=0; x < totals.length; x++){
          if (totals[x].scores[round].score === "0"){
            typeof(totals[x].pointsPerRound) !== 'undefined' ? totals[x].pointsPerRound[round] = null : totals[x].pointsPerRound = [ null ]
          }else {
            if(totals[x] && totals[x+1]){
              if (totals[x].scores[round].score === totals[x+1].scores[round].score){
                numTie++;
                typeof(totals[x].pointsPerRound) !== 'undefined' ? totals[x].pointsPerRound[round] = points : totals[x].pointsPerRound = [ points ]
              } else {
                typeof(totals[x].pointsPerRound) !== 'undefined' ? totals[x].pointsPerRound[round] = points : totals[x].pointsPerRound = [ points ]
                points = points+numTie;
                numTie = 0;
                points++;
              }
            } else {
              typeof(totals[x].pointsPerRound) !== 'undefined' ? totals[x].pointsPerRound[round] = points : totals[x].pointsPerRound = [ points ]
            }
          }
        }
        return totals;
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

        for(let x =0; x< totals[0].scores.length; x++){
          totals = this.sortAtheltesByScoreByRound(totals,x);
          totals = this.calculatePointsByRound(totals,x);
        }
     //   totals.forEach(item => console.log(item.competitorName+"----"+item.pointsPerRound+"----"+item.points ));


        return totals;
      },
    }
}
</script>
