<template>
  <v-container>
    <v-responsive>
      <div>
        <div class="header-title">
          Population growth per country, 1950 to 2021
        </div>
        <div class="header-subtitle">
          Click on the legend below to filter by continent 👇
        </div>
      </div>

      <br>

      <div class="d-flex">
        <div class="region-title">
          Region
        </div>
        <div class="region-item" v-for="(item, index) in regions" :key="index">
          <div :class="`region-color region-${item.color}`"></div>
          <div>
            {{ item.name }}
          </div>
        </div>
      </div>

      <br>

      <div class="graph-layout">
        <div class="range-container">
          <div class="graph-populate-range">
            <div
              v-for="(item, index) in CalculatePopulateRange"
              :key="index" class="graph-populate-range-value"
              :style="`${index == 0 ? 'margin-left: 50px;' : CalculateMarginBar(index)}`"
            >
              <div>
                {{ item }}
              </div>
              <div :class="`${index == 0 ? 'vertical-line' : 'vertical-line line-plus'}`"></div>
            </div>
          </div>
          
          <div class="country-container">
            <div class="graph-country-container" v-for="(item, index) in ListCountryByYear" :key="index">
              <div class="graph-country-text">
                {{ item.name }}
              </div>
              <div :style="SetGraphCountryWidth(item.populate)" :class="`graph-country-color region-${item.color}`">
                <img :src="`src/assets/tn_${item.name}-flag.gif`" class="img-flag" />
              </div>
              <div class="graph-country-populate">{{ parseInt(item.populate).toLocaleString() }}</div>
            </div>
            <div class="current-year-layout">
              <div class="current-year">
                {{ currentYear }}
              </div>
              <div class="current-populate">
                Total: {{ currentTotal.toLocaleString() }}
              </div>
            </div>
          </div>
        </div>
      </div>
      
      <div class="graph-country-slider">
        <v-btn
          :icon="PlayStatusIcon"
          variant="tonal"
          class="graph-country-button"
          @click="PlayYear()"
        >
        </v-btn>
        <v-slider
          class="graph-slider"
          v-model="currentYear"
          :min="1950"
          :max="2021"
          :step="1"
        ></v-slider>
      </div>
      
      <br>
      <br>
      <br>

      <div>
        <div>
          The graph below, this is make from ApexChart library.
        </div>
        <apexchart width="1000" type="bar" :options="ReturnCountry" :series="ReturnPopulate"></apexchart>
      </div>
    </v-responsive>
  </v-container>
</template>

<script>
export default {
  data() {
    return {
      regions: [
        { name: 'Asia', color: 'blue', },
        { name: 'Europe', color: 'purple', },
        { name: 'Africa', color: 'pink', },
        { name: 'Oceania', color: 'orange', },
        { name: 'Americas', color: 'yellow', },
      ],
      notCountRegions: [
        'World', 'Less developed regions', 'Less developed regions, excluding least developed countries', 'Asia (UN)', 'Less developed regions, excluding China',
        'Upper-middle-income countries', 'More developed regions', 'Lower-middle-income countries', 'High-income countries', 'Europe (UN)', 'Africa (UN)',
        'Least developed countries', 'Latin America and the Caribbean (UN)', 'Northern America (UN)', 'Low-income countries', 'Land-locked developing countries (LLDC)'
      ],
      populationData: [],
      currentYear: 1950,
      currentTotal: 0,
      playStatus: false,
      playInterval: null
    }
  },
  mounted() {
    this.GetChartData()
  },
  methods: {
    GetChartData() {
      fetch('https://localhost:7243/api/Populate/getchart', {
        method: 'GET',
        headers: {
          'Content-Type': 'application/json'
        }
      })
      .then((response) => {
        return response.json()
      })
      .then(data => {
        this.populationData = data
      })
      .catch(error => {
        console.log('Looks like there was a problem: \n', error)
      })
    },
    PlayYear() {
      this.playStatus = !this.playStatus
      if (this.playStatus) {
        this.playInterval = setInterval(() => {
          if (this.currentYear >= 2021) {
            this.currentYear = 1950
          } else {
            this.currentYear++
          }
        }, 200);
      } else {
        clearInterval(this.playInterval)
      }
    },
    ReplaceCharacter(string, index, replacement) {
      return (string.slice(0, index) + replacement)
    },
    SetGraphCountryWidth(populate) {
      let width = Math.floor((populate / this.currentTotal) * 100)
      return `width: ${width}vw;`
    },
    CalculateMarginBar(index) {
      if (index == 1) {
        index = 3
      } else if (index == 3) {
        index = 1
      }
      let countryByYear = this.populationData.find(o => o.year == this.currentYear)
      let countryList = []
      if (countryByYear) {
        countryList = countryByYear.countryList.filter(o => this.notCountRegions.indexOf(o.name) === -1).sort((a, b) => b.populate - a.populate)
      }
      let marginLeft = Math.floor(((countryList[index].populate / this.currentTotal) * 450) * 18.5)
      if (this.currentTotal > 0) {
        return `margin-left: ${marginLeft}px;`
      }
      return 'margin-left: 1px;'
    }
  },
  computed: {
    ReturnCountry() {
      let options = {
        chart: {
          id: 'vuechart',
          type: 'bar'
        },
        plotOptions: {
          bar: {
            horizontal: true,
            distributed: true,
            dataLabels: {
              position: 'top'
            },
          }
        },
        colors: [],
        xaxis: {
          categories: []
        }
      }
      let randomColor = ['#6047ec', '#a66de6', '#b7658c', '#db7c30', '#fecc3e']
      let countryByYear = this.populationData.find(o => o.year == this.currentYear)
      if (countryByYear) {
        let countryList = countryByYear.countryList.filter(o => this.notCountRegions.indexOf(o.name) === -1).sort((a, b) => b.populate - a.populate)
        for (let i = 0; i < 12; i++) {
          options.xaxis.categories.push(countryList[i].name)
          let randomNumber = Math.floor(Math.random() * randomColor.length)
          options.colors.push(randomColor[randomNumber])
        }
      }
      return options
    },
    ReturnPopulate() {
      let series = [{
        name: 'series-1',
        data: []
      }]
      let countryByYear = this.populationData.find(o => o.year == this.currentYear)
      if (countryByYear) {
        let countryList = countryByYear.countryList.filter(o => this.notCountRegions.indexOf(o.name) === -1).sort((a, b) => b.populate - a.populate)
        for (let i = 0; i < 12; i++) {
          series[0].data.push(countryList[i].populate)
        }
      }
      return series
    },
    ListCountryByYear() {
      let sum = 0
      let countryByYear = this.populationData.find(o => o.year == this.currentYear)
      let result = []
      let randomColor = ['blue', 'purple', 'pink', 'orange', 'yellow']
      if (countryByYear) {
        let countryList = countryByYear.countryList.filter(o => this.notCountRegions.indexOf(o.name) === -1).sort((a, b) => b.populate - a.populate)
        for (let i = 0; i < 12; i++) {
          let randomNumber = Math.floor(Math.random() * randomColor.length)
          result.push({
            name: countryList[i].name,
            populate: countryList[i].populate,
            color: randomColor[randomNumber]
          })
          sum += parseInt(countryList[i].populate)
        }
        this.currentTotal = sum
      }
      return result 
    },
    PlayStatusIcon() {
      if (this.playStatus) {
        return 'mdi-pause-circle'
      }
      return 'mdi-play-circle'
    },
    CalculatePopulateRange() {
      let countryByYear = this.populationData.find(o => o.year == this.currentYear)
      let result = [ 0 ]
      if (countryByYear) {
        let countryList = countryByYear.countryList.filter(o => this.notCountRegions.indexOf(o.name) === -1).sort((a, b) => b.populate - a.populate)
        for (let i = 0; i >= 0; i--) {
          let tmpText = countryList[i].populate
          if (i == 0) {
            for (let ii = 1; ii < countryList[i].populate.length; ii++) {
              tmpText = this.ReplaceCharacter(tmpText, ii, 0)
            }
          }
          result.push(parseInt(tmpText).toLocaleString())
        }
      }
      return result 
    }
  }
}
</script>
