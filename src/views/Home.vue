<template>
  <div class="home" style="display: flex; flex-direction: column; align-items: center;">
    <h1 style="font-size: 60px;">Heroes of the Gløs</h1>

    <p v-if="!isDataLoaded">Loading data, hold on...</p>

    <div v-else style="display: flex; flex-direction: column; align-items: center; width: 100%;" class="no-cursor">
      <div id="big-number-container" class="more-shadow-box">
        <div class="different-sized-text">
          <span class="big-number">{{totalStats.games}} </span> <p>games</p>
        </div>
        <div class="different-sized-text">
          <span class="big-number">{{totalStats.winRate * 100}}% </span> <p>win rate</p>
        </div>
      </div>

    <button v-if="!isAddingGame" @click="isAddingGame = true" style="margin: 10px 0 0 0; width: fit-content;" class="seethrough-button">
      Add game
    </button>

    <div v-else id="upload" class="more-shadow-box" style="margin-top: 30px;">
      <p><b>Game data</b></p>
      <table id="newGameFirstTable">
        <tr class="no-hover">
          <td colspan="2" style="text-align: center;">
            <input type="radio" v-model="newGame.result" :value="1" id="winYes"/>
            <label for="winYes">Victory</label>
            <input type="radio" v-model="newGame.result" :value="0" id="winNo" style="margin-left: 10px;"/>
            <label for="winNo">Defeat</label>
          </td>
        </tr>

        <tr class="no-hover">
          <td>Map</td>
          <td>
            <select v-model="newGame.map">
              <option v-for="map in legalValues.maps" :key="map" :value="map">{{map}}</option>
            </select>
          </td>
        </tr>

        <tr class="no-hover">
          <td>Game type</td>
          <td>
            <select v-model="newGame.gameType">
              <option v-for="gameType in legalValues.gameTypes" :key="gameType" :value="gameType">{{gameType}}</option>
            </select>
          </td>
        </tr>

        <tr class="no-hover">
          <td>Date</td>
          <td>
            <input type="datetime-local" :value="newGame.date"/>
          </td>
        </tr>

        <tr class="no-hover">
          <td>Password</td>
          <td>
            <input type="text" v-model="newGamePassword" style="width: 100px;"/>
          </td>
        </tr>
      </table>

      <p style="margin-top: 12px;"><b>Players</b></p>
      <table>
        <tr v-for="(player, index) in newGame.team" :key="index" class="no-hover">
          <td>
            Name: <input type="text" v-model="player.name" style="width: 100px;"/>
          </td>
          <td>
            Hero:
            <select v-model="player.hero">
              <option v-for="hero in legalValues.heroes" :key="hero" :value="hero">{{hero}}</option>
            </select>
          </td>
          <td>
            K: <input type="number" v-model="player.kills" style="width: 45px;"/>
          </td>
          <td>
            A: <input type="number" v-model="player.assists" style="width: 45px;"/>
          </td>
          <td>
            D: <input type="number" v-model="player.deaths" style="width: 45px;"/>
          </td>
          <td>
            Award:
            <select v-model="player.award">
              <option v-for="award in legalValues.awards" :key="award" :value="award">{{award}}</option>
            </select>
          </td>
          <td>
            <button @click="removePlayer(index)" style="margin-left: 10px;">x</button>
          </td>
        </tr>
      </table>
      <button @click="addPlayer" style="margin-top: 5px;">+ Add player</button>
      <div>
        <button @click="submitGame" style="margin-top: 15px;">Submit game</button>
        <button @click="isAddingGame = false" style="margin-top: 15px; margin-left: 15px;">Cancel</button>
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
        <p style="font-size: 11px; font-style: italic; margin: auto;">Det kommer mer her senere sånn du kan expande</p>
        <table class="scrolling-table">
          <thead>
            <tr class="no-hover">
              <th>Name</th>
              <th>Games</th>
              <th>Win %</th>
              <th>MVP %</th>
              <th>Award %</th>
              <th>K A D</th>
              <th>K/D</th>
              <th>KA/D</th>
              <th>Fav heroes</th>
            </tr>
          </thead>
          <tr v-for="player in playerStats" :key="player.name">
            <td><p @click="selectPlayer(player)" class="player-name">
              <b>{{player.name}}</b>
            </p></td>
            <td>{{player.games}}</td>
            <td><b>{{player.winRate}}%</b></td>
            <td>{{player.mvpPercentage}}%</td>
            <td>{{player.awardPercentage}}%</td>
            <td>{{player.avgKills}} / {{player.avgAssists}} / {{player.avgDeaths}}</td>
            <td>{{player.avgKD}}</td>
            <td>{{player.avgKAD}}</td>
            <td>
              <table id="fav-heroes-table">
                <tr v-for="hero in favHeroes(player.heroes)" :key="hero.name">
                  <td style="padding: 0">{{hero.name}}</td>
                  <td style="padding: 0 0 0 8px;">{{hero.games}} games</td>
                  <td style="padding: 0 0 0 8px;">{{hero.winRate}}% win</td>
                </tr>
              </table>
            </td>
          </tr>
        </table>
      </div>

      <div v-if="selectedPlayer != null" class="total-stats-container shadow-box" style="color: black;">
        <div style="display: flex; flex-direction: row; justify-content; center; margin: auto; width: fit-content;">
          <h3 style="margin: 0;">{{selectedPlayer.name}} details</h3>
          <button @click="selectPlayer(null)" class="seethrough-button-dark" style="width: fit-content; height: fit-content; margin-left: 20px;">
            Close
          </button>
        </div>
        <table class="scrolling-table">
          <tr>
            <td>
              {{selectedPlayer}}
            </td>
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
// @ is an alias to /src
export default {
  name: 'Home',

  data: function () {
    return {
      isAddingGame: false,
      isDataLoaded: false,
      totalStats: [],
      playerStats: [],
      recentGames: [],
      selectedPlayer: null,
      newGame: {
        'result': undefined,
        'map': undefined,
        'date': undefined,
        'gameType': undefined,
        'team': [this.createEmptyPlayer('Malann'), this.createEmptyPlayer()],
      },
      newGamePassword: '',
      legalValues: {
        'maps': [], 'gameTypes': [], 'heroes': [], 'awards': []
      }
    }
  },

  methods: {
    selectPlayer (player) {
      this.selectedPlayer = player
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
        'gameData': this.newGame,
        'password': this.newGamePassword
      }
      console.log(this.newGamePassword)
      let result = await fetch('/games', {
        method: 'POST',
        headers: {'Content-Type': 'application/json'},
        body: JSON.stringify(requestBody)
      })
      result = await result.text()

      alert(result)
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
    let allGamesReq = await fetch('/games')
    let allGames = await allGamesReq.json()

    this.totalStats = allGames.totalStats
    this.playerStats = allGames.playerStats
    this.recentGames = allGames.recentGames
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
  }
}
</script>

<style lang="scss">
$color1: #22c1c3;
$color2: #fdbb2d;

body {
  background: #FC5C7D;  /* fallback for old browsers */
  background: -webkit-linear-gradient(to bottom right, #6A82FB, #FC5C7D);  /* Chrome 10-25, Safari 5.1-6 */
  background: linear-gradient(to bottom right, #6A82FB, #FC5C7D); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */

  background: -webkit-linear-gradient(to bottom right, #dd3e54, #6be585);  /* Chrome 10-25, Safari 5.1-6 */
  background: linear-gradient(to bottom right, #dd3e54, #6be585); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */

  background: -webkit-linear-gradient(to bottom right, #22c1c3, #fdbb2d);  /* Chrome 10-25, Safari 5.1-6 */
  background: linear-gradient(to bottom right, #22c1c3, #fdbb2d); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */

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
}

.scrolling-table {
  width: 100%;
}

.win-game {
  border: 1px solid $color1;
}
.lose-game {
  border: 1px solid $color2;
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

table {
  &:hover {
    cursor: default;
  }
  width: fit-content;
  border-collapse: collapse;
  display: block;
  overflow-x: auto;
  white-space: nowrap;
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
    background-color: #eaeaea;
    box-shadow: 0 3px 6px rgba(0,0,0,0.16), 0 3px 6px rgba(0,0,0,0.23);
    &.button-disabled {
      box-shadow: 0 1px 3px rgba(0,0,0,0.12), 0 1px 2px rgba(0,0,0,0.24);
    }
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
  background-color: rgba(0,0,0, 0.55);
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
  td {
    text-align: left;
    &:first-child {
      text-align: right;
      padding-right: 8px;
    }
  }
}

.player-name {
  text-decoration: underline;
  text-decoration-color: $color1;
  &:hover {
    cursor: pointer;
  }
}

table {
  border-collapse: collapse;
  td {
    text-align: center;
  }
}

#fav-heroes-table {
  td {
    text-align: left !important;
  }
}
</style>