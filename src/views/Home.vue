<template>
  <div class="home dark" style="display: flex; flex-direction: column; align-items: center;">
    <h1>Heroes of the Gløs</h1>

    <p v-if="!isDataLoaded">Loading data, hold on...</p>

    <div v-else style="display: flex; flex-direction: column; align-items: center; width: 100%;" class="no-cursor">
      <div id="big-number-container" class="more-shadow-box">
        <div class="different-sized-text">
          <span class="big-number">{{totalStats.games}} </span> <p>games</p>
        </div>
        <div class="different-sized-text">
          <span class="big-number">{{(Math.round(totalStats.winRate*1000))/10}}% </span> <p>win rate</p>
        </div>
      </div>

    <button v-if="!isAddingGame" @click="isAddingGame = true" style="margin: 10px 0 0 0; width: fit-content;" class="seethrough-button">
      Add game
    </button>

    <div v-else id="upload" class="more-shadow-box" style="margin-top: 30px;">
      <p><b>New game</b></p>
      <div id="newGameFirstTable">
        <div style="display: flex; gap: 1.5rem; justify-content: center; margin-top: 0.5rem; margin-bottom: -0.5rem;">
          <Radio label="Victory" :currentValue="newGame.result" value="1"  @onChange="onResultChange"/>
          <Radio label="Defeat" :currentValue="newGame.result" value="0"  @onChange="onResultChange"/>
        </div>

        <Select :options="legalValues.maps.map(map => ({text: map, value: map}))"
                title="Map"
                isSearchable
                searchPlaceholder="Map"
                @change="onMapSelect"
                @searchSelectedClicked="() => newGame.map = undefined"
                :resetValue="mapResetValue"
                :searchSelected="newGame.map"
                isFullWidth
                style="width: 15rem;"/>

        <Select :options="legalValues.gameTypes.map(gt => ({text: gt, value: gt}))"
                title="Game type"
                @change="onGameTypeChange"
                :resetValue="inputsResetValue"
                isFullWidth/> 

        <DateTimePicker :value="newGame.date"
                        @change="onDateChange"
                        title="Date &amp; time"/>

        <TextInput @change="newVal => newGamePassword = newVal"
                    title="Password"
                    textAlign="left"
                    style="width: 100%;"/>
      </div>

      <p style="margin-top: 12px;">
        <b>Players</b>
      </p>
      <div style="margin: 0.5rem 0; display: flex; flex-direction: column; gap: 0.75rem;">
        <div v-for="(player, index) in newGame.team" :key="index" style="display: flex; gap: 0.5rem; align-items: flex-end;">
          <TextInput @change="newVal => player.name = newVal"
                      title="Name"
                      style="width: 9rem;"
                      textAlign="left"/>
          <Select :options="legalValues.heroes.map(hero => ({text: hero, value: hero}))"
                  title="Hero"
                  isSearchable
                  @change="newVal => {
                    player.hero = newVal;
                    heroResetValue = Math.random().toString()
                  }"
                  style="width: 11rem;"
                  @searchSelectedClicked="() => player.hero = undefined"
                  :resetValue="heroResetValue"
                  :searchSelected="player.hero"/>
          <TextInput @change="newVal => player.kills = newVal"
                      title="K"
                      type="number"
                      style="width: 3rem;"
                      textAlign="left"/>
          <TextInput @change="newVal => player.assists = newVal"
                      title="A"
                      type="number"
                      style="width: 3rem;"
                      textAlign="left"/>
          <TextInput @change="newVal => player.deaths = newVal"
                      title="D"
                      type="number"
                      style="width: 3rem;"
                      textAlign="left"/>
          <Select :options="legalValues.awards.map(award => ({text: award, value: award}))"
                  title="Award"
                  isSearchable
                  @change="newVal => {
                    player.award = newVal;
                    heroResetValue = Math.random().toString()
                  }"
                  style="width: 11rem;"
                  @searchSelectedClicked="() => player.award = undefined"
                  :resetValue="heroResetValue"
                  :searchSelected="player.award"/>
          <button @click="removePlayer(index)" style="margin-left: 10px;" class="iconButton">
            <CloseIcon title=""/>
          </button>
        </div>
      </div>
      <button @click="addPlayer" style="margin-top: 0.5rem;">
        + Add player
      </button>
      <div style="display: flex; flex-direction: row; margin-top: 1.5rem; gap: 1rem;">
        <button @click="cancelAddingGame()">
          Cancel
        </button>
        <button @click="submitGame">
          Submit game
        </button>
      </div>
    </div>

      <div id="major-stats-container" style="width: 100%;">
        <div class="total-stats-container shadow-box">
          <h3>Wins by game type</h3>
          <table class="scrolling-table">
            <tr v-for="(data, gameType) in totalStats.winRatesByGameType" :key="gameType">
              <td>{{gameType}}</td>
              <td><span class="different-sized-text">
                <span class="medium-number">{{data.games || '-'}}</span> <p>games</p>
              </span></td>
              <td><span class="different-sized-text">
                <span class="medium-number">{{percentToString(data.winRate)}}</span> <p>win rate</p>
              </span></td>
            </tr>
          </table>
        </div>

        <div class="total-stats-container shadow-box">
          <h3>Wins by map</h3>
          <table class="scrolling-table">
            <tr v-for="mapData in totalStats.winRatesByMap" :key="mapData.map">
              <td style="text-align: left;">{{mapData.map}}</td>
              <td><span class="different-sized-text">
                <span class="medium-number">{{mapData.games}}</span> <p>games</p>
              </span></td>
              <td><span class="different-sized-text">
                <span class="medium-number">{{percentToString(mapData.winRate)}}</span> <p>win rate</p>
              </span></td>
            </tr>
          </table>
        </div>

        <div class="total-stats-container shadow-box">
          <h3>Wins by # of players</h3>
          <table class="scrolling-table">
            <tr v-for="num in [2,3,4,5]" :key="num">
              <td>{{num}} players</td>
              <td><span class="different-sized-text">
                <span class="medium-number">{{totalStats.winratesByPlayerCounts[num].games || '-'}}</span> <p>games</p>
              </span></td>
              <td><span class="different-sized-text">
                <span class="medium-number">{{percentToString(totalStats.winratesByPlayerCounts[num].winRate)}}</span> <p>win rate</p>
              </span></td>
            </tr>
          </table>
        </div>
      </div>

      <div class="total-stats-container shadow-box" id="player-table-container">
        <h3>Player stats</h3>
        <table class="scrolling-table">
          <thead>
            <tr class="no-hover">
              <th>Name</th>
              <th>#</th>
              <th style="padding-left: 12px;">Win %</th>
              <th>MVP %</th>
              <th>Award %</th>
              <th>K A D</th>
              <th>K/D</th>
              <th>KA/D</th>
              <th>Fav heroes</th>
            </tr>
          </thead>
          <tr v-for="player in playerStats" :key="player.name">
            <td><p @click="selectPlayer(player)" class="clickable">
              <b>{{player.name}}</b>
            </p></td>
            <td>{{player.games}}</td>
            <td style="padding-left: 12px;"><b>{{player.winRate}}%</b> ({{player.wins}})</td>
            <td>{{player.mvpPercentage}}%</td>
            <td>{{player.awardPercentage}}%</td>
            <td>{{player.avgKills}} / {{player.avgAssists}} / {{player.avgDeaths}}</td>
            <td>{{player.avgKD}}</td>
            <td>{{player.avgKAD}}</td>
            <td>
              <table id="fav-heroes-table">
                <tr v-for="hero in favHeroes(player.heroes)" :key="hero.name" class="no-hover">
                  <td style="padding: 0">{{hero.name}}</td>
                  <td style="padding: 0 0 0 8px;">{{hero.games}} games</td>
                  <td style="padding: 0 0 0 8px;">{{hero.winRate}}% win</td>
                </tr>
              </table>
            </td>
          </tr>
        </table>
      </div>

      <div v-if="selectedPlayer != null" class="total-stats-container shadow-box" id="selected-player-stats">
        <div style="display: flex; flex-direction: row; justify-content; center; margin: 0 auto 0 auto; width: fit-content;">
          <h3 style="margin: 0;">{{selectedPlayer.name}} details</h3>
          <button @click="selectPlayer(null)" class="seethrough-button-dark" style="width: fit-content; height: fit-content; margin-left: 20px;">
            Close
          </button>
        </div>

        <div style="display: flex; flex-direction: row; flex-wrap: wrap; justify-content: center;">
          <div style="margin: 10px">
            <h4>All hero stats</h4>
            <table class="scrolling-table">
              <thead>
                <tr class="no-hover">
                  <th>Hero</th>
                  <th style="padding-right: 4px;">#</th>
                  <th style="padding-left: 4px;">Win %</th>
                  <th>K A D</th>
                </tr>
              </thead>
              <tr v-for="hero in selectedPlayer.heroes" :key="hero.name">
                <td style="text-align: left;">{{hero.name}}</td>
                <td>{{hero.games}}</td>
                <td>{{hero.winRate}}%</td>
                <td>{{hero.avgKills}} / {{hero.avgAssists}} / {{hero.avgDeaths}}</td>
              </tr>
            </table>
          </div>

          <div style="margin: 10px;">
            <h4>Awards</h4>
            <table class="scrolling-table">
              <tr v-for="award in selectedPlayer.awards" :key="award.award">
                <td style="text-align: left;">{{award.award}}</td>
                <td>{{award.percentage}}%</td>
              </tr>
            </table>
          </div>

          <div style="margin: 10px;">
            <h4>Roles</h4>
            <p>Coming soon!</p>
          </div>
        </div>
      </div>



      <div class="total-stats-container shadow-box" id="teams-stats">
        <h3>Teams stats</h3>
        <p style="text-align: center; margin: 0 auto 0.5rem auto; font-size: 12px;">
          Click a team to initiate a new game registration with players
        </p>
        <table class="scrolling-table" style="margin: auto;">
          <thead>
            <tr class="no-hover">
              <th>Players</th>
              <th>Games</th>
              <th style="padding-left: 8px;">Win %</th>
            </tr>
          </thead>
          <tr v-for="team in teamStats" :key="team.playersString">
            <td style="text-align: left;">
              <p class="teamPlayers clickable clickable-light" @click="initiateNewGameWithPlayers(team.playersString)">
                {{team.playersString}}
              </p>
            </td>
            <td>{{team.games}}</td>
            <td style="padding-left: 8px;">{{team.winPercent}}%</td>
          </tr>
        </table>
      </div>





      <div class="total-stats-container shadow-box" id="recent-games-container">
        <h3>Most recent 10 games</h3>

        <div v-for="(game, index) in recentGames" :key="index" class="one-game" :class="{'lose-game': !game.result, 'win-game': game.result}">
          <div class="title">
            <div style="width: 100%;">
              <p class="no-cursor">{{game.map}} - {{game.result ? 'Victory' : 'Defeat'}}</p>
              <p style="font-size: 12px; margin-left: 2px;" class="no-cursor">
                {{prettyDate(game.date)}}
              </p>
            </div>
          </div>
          <table class="players">
            <tr v-for="player in game.team" :key="player.name" class="no-hover">
              <td>{{player.name}}</td>
              <td>{{player.hero}}</td>
              <td>{{player.kills}} / {{player.assists}} / {{player.deaths}}</td>
              <td>{{player.award || '-'}}</td>
            </tr>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import TextInput from '@/components/TextInput.vue'
import Select from '@/components/Select.vue'
import Radio from '@/components/Radio.vue'
import DateTimePicker from '@/components/DateTimePicker.vue'
import CloseIcon from 'vue-material-design-icons/Close.vue'

export default {
  name: 'Home',

  components: {
    TextInput, Select, Radio, DateTimePicker, CloseIcon,
  },

  data () {
    return {
      isAddingGame: false,
      isDataLoaded: false,
      totalStats: [],
      playerStats: [],
      recentGames: [],
      selectedPlayer: null,
      newGame: this.makeDefaultEmptyGame(),
      newGamePassword: '',
      legalValues: {
        'maps': [], 'gameTypes': [], 'heroes': [], 'awards': []
      },
      inputsResetValue: null,
      mapResetValue: null,
      heroResetValue: null,
    }
  },

  methods: {
    onMapSelect (newMap) {
      this.newGame.map = newMap
      this.mapResetValue = Math.random().toString()
    },

    onGameTypeChange (newVal) {
      this.newGame.gameType = newVal
    },

    onResultChange (newResult) {
      this.newGame.result = newResult
      console.log(newResult)
    },

    onDateChange (newVal) {
      this.newGame.date = newVal
    },

    selectPlayer (player) {
      this.selectedPlayer = player
      setTimeout(() => {
        const element = document.getElementById('selected-player-stats')
        const elementRect = element.getBoundingClientRect()
        const elementCenter = (elementRect.top + elementRect.bottom) / 2
        const absoluteElementCenter = elementCenter + window.pageYOffset
        const middle = absoluteElementCenter - (window.innerHeight / 2)
        // document.getElementById('selected-player-stats').scrollIntoView({behavior: 'smooth'})
        window.scrollTo({
          top: middle,
          behavior: 'smooth'
        })
      }, 10)
    },

    cancelAddingGame () {
      this.isAddingGame = false
      this.newGame.team = [this.createEmptyPlayer('Malann'), this.createEmptyPlayer()]
    },

    createEmptyPlayer (name='') {
      return {
        'name': name,
        'hero': undefined,
        'kills': undefined,
        'assists': undefined,
        'deaths': undefined,
        'award': null,
      }
    },

    initiateNewGameWithPlayers (playersString) {
      this.isAddingGame = true
      this.newGame.team = playersString.split(', ').map(player => {
        return this.createEmptyPlayer(player)
      })
      window.scrollTo(0,0)
    },

    addPlayer () {
      this.newGame.team.push(this.createEmptyPlayer())
    },

    removePlayer (playerIndex) {
      if (this.newGame.team.length > 1) {
        this.newGame.team.splice(playerIndex, 1)
      }
    },

    async submitGame () {
      let requestBody = {
        'gameData': {...this.newGame, result: Number(this.newGame.result)},
        'password': this.newGamePassword
      }
      console.log("Passordet er 'Orphea' lol")

      let result = await fetch('/games', {
        method: 'POST',
        headers: {'Content-Type': 'application/json'},
        body: JSON.stringify(requestBody)
      })

      result = await result.text()
      alert(result)
      if (result.includes('Success')) { // LOL.
        this.isAddingGame = false
        this.newGame = this.makeDefaultEmptyGame()
        this.mapResetValue = Math.random().toString()
        this.heroResetValue = Math.random().toString()
        this.inputsResetValue = Math.random().toString()
        this.fetchData()
      }

    },

    favHeroesToStrings (heroes) {
      let strings = []
      for (let i = 0; (i < 3 && i < heroes.length); i++) {
        strings.push(`${heroes[i].name}: ${heroes[i].games} games, ${heroes[i].winRate}% win`)
      }
      return strings
    },

    favHeroes (heroes) {
      return heroes.slice(0,3)
    },
    
    async fetchData () {
      let allGamesReq = await fetch('/games')
      let allGames = await allGamesReq.json()

      this.totalStats = allGames.totalStats
      this.playerStats = allGames.playerStats
      this.recentGames = allGames.recentGames
      this.teamStats = allGames.teamStats.sort((ts1, ts2) => ts1.games > ts2.games ? -1 : 1)

      for (let player of this.playerStats) {
        player.isExpanded = false
      }
      
      for (let map of this.totalStats.winRatesByMap) {
        if (map.map === 'Tomb of the Spider Queen') {
          map.map = 'Tomb ot Spider Queen'
        }
      }

      this.isDataLoaded = true

      let theLegalValuesReq = await fetch('/legal-values')
      this.legalValues = await theLegalValuesReq.json()

      this.newGame.date = (new Date()).toISOString().substr(0,16)
    },

    makeDefaultEmptyGame () {
      return {
        'result': '',
        'map': undefined,
        'date': undefined,
        'gameType': undefined,
        'team': [this.createEmptyPlayer('Malann'), this.createEmptyPlayer()],
      }
    },

    percentToString (percentNumber) {
      if (percentNumber == null || percentNumber == undefined) {
        return '-'
      }
      else {
        return percentNumber + '%'
      }
    },

    prettyDate (date) {
      let d = new Date(date)
      return d.toDateString().substr(0,10) + ", " + d.toTimeString().substr(0,5)
    }
  },

  async mounted () {
    this.fetchData()
  }
}
</script>

<style lang="scss">
@import "../scss/colors.scss";

input, select {
  margin: 0;
  padding: 0;
}

.body-fall {
  background: #FC5C7D;  /* fallback for old browsers */
  background: -webkit-linear-gradient(to bottom right, #f55353, #e9c439);  /* Chrome 10-25, Safari 5.1-6 */
  background: linear-gradient(to bottom right, #f55353, #e9c439); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */
}

.body-winter {
  background: $seasonColor1;  /* fallback for old browsers */
  background: -webkit-linear-gradient(to bottom right, $seasonColor1, #9773eb, $seasonColor2);  /* Chrome 10-25, Safari 5.1-6 */
  background: linear-gradient(to bottom right, $seasonColor1, #9773eb, $seasonColor2); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */
}

.body-spring {
  background: #22c1c3;  /* fallback for old browsers */
  background: -webkit-linear-gradient(to bottom right, #22c1c3, #fdbb2d);  /* Chrome 10-25, Safari 5.1-6 */
  background: linear-gradient(to bottom right, #22c1c3, #fdbb2d); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */
}

.cursorPointer {
  &:hover {
    cursor: pointer;
  }
}

h1, h2, h3, h4, p {
  margin: 0; padding: 0;
  width: fit-content;
}
h1 {
  font-family: 'Arvo', serif;
  margin: auto;
  padding-top: 10px;
  text-align: center;
  font-size: 4rem;
  @media (max-width: 860px) {
    font-size: 2.25rem;
    margin-top: 1.5rem;
    margin-bottom: 1rem;
  }
}

input {
  font-family: 'Montserrat', sans-serif;
  font-size: 1rem;
}

.scrolling-table {
  width: 100%;
  overflow-x: auto;
}

.teamPlayers {
  text-align: left;
  &:hover {
    cursor: pointer;
  }
}

.win-game {
  border: 1px solid $theme1;
}
.lose-game {
  border: 1px solid $theme2;
}

.one-game {
  margin: 8px auto;
  background: rgba(255,255,255, 0.15);
  padding: 4px;

  &:hover {
    background: rgba(255,255,255, 0.35);
  }
  .players {
    font-size: 12px;
    margin-left: 6px;
    td {
      text-align: left;
    }
  }
}
tr:not(.no-hover) {
  &:hover {
    background: rgba(255,255,255, 0.35);
  }
}
.no-cursor {
  cursor: default;
  &:hover {
    cursor: default;
  }
}

#teams-stats {
  color: black;
  td {
    padding: 4px 0;
  }
}

#selected-player-stats {
  color: black;
  h4 {
    margin-bottom: 4px;
  }
}

table {
  &:hover {
    cursor: default;
  }
  width: fit-content;
  border-collapse: collapse;
  display: block;
  white-space: nowrap;
}

.iconButton {
  border-radius: 40px;
  padding: 0.5rem;
  width: 2rem;
  height: 2rem;
  span {
    margin-right: 4px;
  }
  svg {
    margin-bottom: 4px;
  }
}

.total-stats-container {
  background: rgba(255, 255, 255, 0.5);
  width: fit-content;
  margin: 30px;
  padding: 10px;
  white-space: nowrap;
  max-width: 100%;
  h3 {
    text-align: center;
    font-size: 26px;
    font-weight: 300;
    margin: 0 auto 5px auto;
  }
  td {
    padding-right: 10px;
  }
}

h4 {
  font-size: 20px;
  font-weight: 400;
  margin: auto;
}

.material-design-icon { 
  display: inline-flex;
  align-self: center;
  position: relative;
  height: 1rem;
  width: 1rem;
}

.material-design-icon>.material-design-icon__svg {
  height: 1rem;
  width: 1rem;
  fill: currentColor;
  position: absolute;
  bottom: -0.125rem;
}

.shadow-box {
  box-shadow: 0 8px 16px 0px rgba(25, 25, 25, 0.12), 0px 8px 64px 0px rgba(25, 25, 25, 0.25);
  &:hover {
    box-shadow: 0 8px 16px 0px rgba(25, 25, 25, 0.20), 0px 10px 70px 0px rgba(25, 25, 25, 0.40);
  }
}
.more-shadow-box {
  box-shadow: 0 8px 16px 0px rgba(25, 25, 25, 0.20), 0px 8px 64px 0px rgba(25, 25, 25, 0.40);
  &:hover {
    box-shadow: 0 8px 16px 0px rgba(25, 25, 25, 0.28), 0px 10px 78px 0px rgba(25, 25, 25, 0.48);
  }
}

.seethrough-button {
  background-color: transparent;
  color: #f5f5f5;
  font-size: 16px;
  border: 1px solid #ddd;
  &:hover {
    background-color: transparent;
  }
}

.seethrough-button-dark {
  background-color: rgba(111,111,111,0.15);
  color: #333;
  font-size: 16px;
  margin-top: 2px;
  border: none;
  &:hover {
    background-color: transparent;
  } 
}

#upload {
  background-color: rgba(0,0,0, 0.75);
  color: white;
  width: fit-content;
  margin: auto;
  margin-bottom: 20px;
  padding: 20px 15px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

#major-stats-container {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: flex-start;
  color: black;
  flex-wrap: wrap;
}

.big-number {
  font-size: 50px;
  font-weight: 600;
}

#big-number-container {
  margin: 20px auto 20px auto;
  background: rgba(222, 222, 222, 0.3);
  padding: 10px 30px;
  font-size: 24px;
  color: white;
  width: fit-content;
  display: flex;
  flex-direction: column;
  align-items: center;
  div {
    display: flex;
    justify-content: center;
    p {
      margin-left: 8px;
    }
  }
}

#player-table-container {
  background-color: rgba(255, 255, 255, 0.5);
  color: black;
  margin: 30px auto;
  table {
    td {
      padding: 10px 4px;
    }
    th {
      padding: 0 4px;
    }
    tr:not(.no-hover) {
      &:hover {
        background-color: rgba(255, 255, 255, 0.2);
        // box-shadow: 2px 2px 10px 10px rgba(25, 25, 25, 0.04);
      }
    }
  }
}

#recent-games-container {
  background-color: rgba(255, 255, 255, 0.5);
  margin-right: auto;
  margin-left: auto;
  color: black;
}

.different-sized-text {
  display: flex; flex-direction: row; align-items: center;
  p {
    margin-left: 6px;
  }
}
.medium-number {
  font-weight: 600;
  font-size: 22px;
}
#players-grid {
  display: grid;
  grid-template-columns: auto auto auto auto;
}

#newGameFirstTable {
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
}

.clickable {
  display: inline-block;
  line-height: 0.88;
  border-bottom: 3px dashed $theme1;
  &:hover {
    cursor: pointer;
    border-bottom: 3px solid $theme2;
  }
}

.clickable-light {
  border-bottom-width: 1.5px !important;
}

table {
  border-collapse: collapse;
  td {
    text-align: center;
  }
}

button {
  font-size: 14px;
  box-sizing: border-box;
  border: none;
  padding: 4px 8px;
  border-radius: 4px;
  background-color: #fff;
  color: black;
  box-shadow: 0 1px 3px rgba(0,0,0,0.12), 0 1px 2px rgba(0,0,0,0.24);

  &:hover {
    cursor: pointer;
    box-shadow: 0 3px 6px rgba(0,0,0,0.16), 0 3px 6px rgba(0,0,0,0.23);
    &.button-disabled {
      box-shadow: 0 1px 3px rgba(0,0,0,0.12), 0 1px 2px rgba(0,0,0,0.24);
    }
  }
}

button {
  background: rgba(255, 255, 255, 0.9);
  color: #000;
  font-family: 'Montserrat', sans-serif;
  font-size: 1rem;
  font-weight: 400;
  padding: 0.5rem 1rem;
  border-radius: 6px;
  display: flex;
  align-items: center;
  box-shadow: 0 4px 10px 0 rgba(25, 25, 25, 0.06), 0 4px 16px 0 rgba(25, 25, 25, 0.15);
  span {
    margin-bottom: -5px;
    margin-right: 0.3rem;
  }
  
  background: transparentize($color: $theme1, $amount: 0.5);
  color: white;
  &:hover {
    background: transparentize($color: $theme1, $amount: 0.2);
    box-shadow: 0 8px 16px 0 rgba(25,25,25,.12), 0 8px 32px 0 rgba(25,25,25,.25);
  }

  transition: all 100ms;
}

#fav-heroes-table {
  td {
    text-align: left !important;
  }
}
</style>