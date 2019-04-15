<template>
  <div>
    <svg class="specialty-chart" ref="specialty"></svg>
    <div class="internal-medicine">
      Doctors working in
      <span class="selected-medicine dropdown-container">
        <b-nav-item-dropdown :text="selectedMed" right>
            <b-dropdown-item href="#" :key="index"
                v-on:click="selectMedicine(med)"
                v-for="(med, index) of internalMeds" >
                {{med}}
            </b-dropdown-item>
        </b-nav-item-dropdown>
      </span>
      in
      <span class="selected-state dropdown-container">
        <b-nav-item-dropdown :text="selectedState.name" right>
          <b-dropdown-item href="#" :key="index"
              v-on:click="selectState(state)"
              v-for="(state, index) of states">
              {{state.name}}
          </b-dropdown-item>
        </b-nav-item-dropdown>
      </span>
      prescribe
      <span class="num-prescriptions dropdown-container">
        <b-nav-item-dropdown text="100000" right>
            <b-dropdown-item href="#">EN</b-dropdown-item>
            <b-dropdown-item href="#">ES</b-dropdown-item>
            <b-dropdown-item href="#">RU</b-dropdown-item>
            <b-dropdown-item href="#">FA</b-dropdown-item>
        </b-nav-item-dropdown>
      </span>
      opioid prescriptions
      per year
    </div>
  </div>
</template>

<script>
// external libraries
import * as d3 from 'd3'
import * as _ from 'lodash'

// read chart data
import data_2013 from '@/assets/data/2013.json'
import data_2014 from '@/assets/data/2014.json'
import data_2015 from '@/assets/data/2015.json'
import data_2016 from '@/assets/data/2016.json'

import STATES from '@/assets/states.json'

// init chart constants
const WIDTH = 1200
const HEIGHT = 600
const MARGIN = {
  top: 20,
  bottom: 60,
  left: 20,
  right: 20
}
const SUB_HEADING_SHIFT = 150
const HEADING_SHIFT = 100
const headingWidth = WIDTH / 2 - 300
const heading = 'How Does the United States Feel Pain?'
const DATA = {
  2013: data_2013,
  2014: data_2014,
  2015: data_2015,
  2016: data_2016
}
const INITYEAR = 2013
const DATA_RANGE = 45
const YEARS = _.range(2013, 2017)

// chart vue code
export default {
  name: 'SpecialtyChart',
  data: () => {
    return {
      year: INITYEAR,
      data: [],
      internalMeds: [],
      states: [],
      selectedMed: '',
      selectedState: {},
      svgElement: '',
      svg: '',
      chartg: '',
      width: WIDTH - MARGIN.left - MARGIN.right,
      height: HEIGHT - MARGIN.top - MARGIN.bottom,
      yscale: '',
      xscale: '',
      yearScale: ''
    }
  },
  mounted() {
      this.changeYearData();
      this.svgElement = this.$refs.specialty;
      this.initChart();
      this.drawChart();
      this.addYearAxis();
  },
  methods: {
    initChart: function() {
      this.svg = d3.select(this.svgElement)
        .attr('width', WIDTH)
        .attr('height', HEIGHT)

      // chart heading
      this.svg.append('text')
        .classed('specialty-chart-heading', true)
        .attr('x', headingWidth)
        .attr('y', HEADING_SHIFT)
        .text(heading);

      //main chart container
      this.chartg = this.svg
        .append('g')
        .attr('transform', `translate(${MARGIN.left}, ${MARGIN.top + SUB_HEADING_SHIFT})`)

      // x scale
      this.xscale = d3.scalePoint()
        .range([0, this.width])
        .domain(_.range(0, DATA_RANGE))

      // y scale. shift height due to heading and sub heading
      this.yscale = d3.scaleLinear()
        .range([this.height - (MARGIN.top + SUB_HEADING_SHIFT), 0])
        .domain([0, 100])

      this.yearScale = d3.scalePoint()
        .range([0.2 * this.width, 0.8 * this.width])
        .domain(YEARS)
    },
    addYearAxis() {
      const axis = d3.axisBottom(this.yearScale);
      this.chartg.append('g')
        .attr('class', 'year-axis')
        .attr('transform', `translate(0, ${this.height - (MARGIN.top + SUB_HEADING_SHIFT - 30)})`)
        .call(axis)

      const self = this;

      d3.selectAll('.tick text')
        .classed('highlighted', function(d) {
          const year = +d3.select(this).text();
          if(year === INITYEAR) {
            return true
          } else {
            return false
          }
        })
        .on('click', function(d) {
          self.year = +d3.select(this).text();
          d3.selectAll('.tick text').classed('highlighted', false)
          d3.select(this).classed('highlighted', true)
          self.changeYearData();
          self.drawChart();
        });
    },
    selectState(state) {
      this.selectedState = state; //{ state: state, name: STATES[state] };
      this.drawChart();
    },
    selectMedicine(med) {
      this.selectedMed = med;
      this.drawChart();
    },
    changeYearData() {
      this.data = DATA[this.year];
      const chainData = _.chain(this.data);

      this.internalMeds = chainData.map('specialty_description')
          .uniq()
          .concat('All')
          .sortBy()
          .value()

      this.states = chainData.map('nppes_provider_state').uniq()
        .concat('ALL')
        .sortBy()
        .map(state => {
          return { state: state, name: STATES[state] }
        })
        .value()

      if(_.keys(this.selectedState).length === 0)
        this.selectedState = this.states.filter(d => d.state === 'ALL')[0]
      if(this.selectedMed === '')
        this.selectedMed = this.internalMeds.filter(d => d === 'All')[0]
    },
    filterData: function() {
      // filter the data to get the relevant dataset
      const filtered = _.chain(this.data)
          .filter(d => (this.selectedMed === 'All' ||
                d.specialty_description === this.selectedMed) &&
              (this.selectedState.state === 'ALL' ||
                d.nppes_provider_state === this.selectedState.state)
          )
          // sort by the sum of opioid_prescriber_rate and la_opioid_prescriber_rate ascending
          .sortBy(d => (+d.opioid_prescriber_rate || 0) + (+d.la_opioid_prescriber_rate || 0))
          // make sure values are numbers
          .map(d => {
            _.keys(d).forEach(key => {
              d[key] = typeof +d[key] === 'undefined' || isNaN(+d[key]) ? d[key] : +d[key];
            })
            if(typeof d.la_opioid_prescriber_rate === 'undefined') {
              d.la_opioid_prescriber_rate = 0
            }
            if(typeof d.opioid_prescriber_rate === 'undefined') {
              d.opioid_prescriber_rate = 0
            }
            return d
          })
          // make it descending
          .reverse()
          // remove undefined / 0 values
          .filter(d => !(d.la_opioid_prescriber_rate === 0 && d.opioid_prescriber_rate === 0))
          // .filter(d => typeof d.la_opioid_prescriber_rate !== 'undefined' &&
          //             typeof d.opioid_prescriber_rate !== 'undefined') // &&
                      // d.opioid_prescriber_rate !== 0 &&
                      // d.la_opioid_prescriber_rate !== 0)
          .slice(0, DATA_RANGE)
          .value()

      return filtered
    },
    drawChart: function() {
      const data = this.filterData()

      const numFormat = d3.format('.0%');

      const gdata = this.chartg.selectAll('.ball-group')
        .data(data, (d, i) => this.year + d.specialty_description + i + d.nppes_provider_state)

      gdata.exit().remove();

      // circle line text group
      const group = gdata.enter()
        .append('g')
        .classed('ball-group', true)
        .attr('transform', (d, i) => {
          return `translate(${this.xscale(i)}, 0)`
        })

      group.on('mouseenter', function(d){
        d3.selectAll('.ball-group').classed('faded', true);
        d3.selectAll('.ball-group').classed('highlighted', false);
        d3.select(this).classed('faded', false);
        d3.select(this).classed('highlighted', true);
      })

      group.on('mouseleave', function(d){
        d3.selectAll('.ball-group').classed('faded', false);
        d3.selectAll('.ball-group').classed('highlighted', false);
      })

      // opioid_prescriber_rate circle
      group.append('circle')
        .filter(d => d.opioid_prescriber_rate !== 0)
        .attr('r', 5)
        .attr('cy', d => {
          const yval = this.yscale(d.opioid_prescriber_rate)
          return yval;
        })
        .classed('opioid-prescriber-rate', true);

      // opioid_prescriber_rate text
      group.append('text')
        .filter(d => d.opioid_prescriber_rate !== 0)
        .attr('x', 10)
        .attr('y', d => this.yscale(d.opioid_prescriber_rate))
        .classed('opioid-prescriber-rate opioid-value', true)
        .text(d => numFormat(d.opioid_prescriber_rate/100))

      // opioid_prescriber_rate circle
      group.append('circle')
        .filter(d => d.la_opioid_prescriber_rate !== 0)
        .attr('r', 5)
        .attr('cy', d => this.yscale(d.la_opioid_prescriber_rate))
        .classed('la-opioid-prescriber-rate', true);

      // opioid_prescriber_rate text
      group.append('text')
        .filter(d => d.la_opioid_prescriber_rate !== 0)
        .attr('x', 10)
        .attr('y', d => this.yscale(d.la_opioid_prescriber_rate))
        .classed('la-opioid-prescriber-rate opioid-value', true)
        .text(d => numFormat(d.la_opioid_prescriber_rate/100))

      // white line joining the circles
      group.append('line')
          .filter(d => d.la_opioid_prescriber_rate !== 0)
          .attr('r', 5)
          .attr('y1', d => this.yscale(d.opioid_prescriber_rate))
          .attr('y2', d => this.yscale(d.la_opioid_prescriber_rate))
          .classed('opioid-prescriber-line', true);
    }
  }
}
</script>

<style lang="scss">
  .year-axis {
    .tick {
      line { stroke: #fff; }
      text {
        fill: #fff;
        cursor: pointer;
        font-size: 12px;
      }
      .highlighted {
        font-size: 16px;
        fill: #F8E368;
        font-weight: bold;
      }
    }
    path {
      stroke: #fff;
    }
  }
  .internal-medicine {
    position: absolute;
    top: 120px;
    left: 50%;
    width: 660px;
    color: #fff;
    font-size: 20px;
    .dropdown-container {
      display: inline-block;
      .nav-item {
        list-style: none;
        a {
          color: #F8E368;
          font-weight: bold;
        }
        .dropdown-menu {
          background-color: #000;
          height: 300px;
          overflow-y: scroll;
          overflow-x: none;
        }
      }
    }
  }
  .specialty-chart-heading {
    font-size: 40px;
    fill: #F8E368;
  }
  .specialty-chart-sub-heading {
    fill: #FFF;
    font-size: 20px;
  }
  .ball-group {
    cursor: pointer;
    .opioid-prescriber-rate {
      fill: #F8E368
    }
    .la-opioid-prescriber-rate {
      fill: #fff;
    }
    .opioid-prescriber-line {
      stroke: #fff;
      fill: none;
    }
    .opioid-value {
      display: none;
    }
  }
  .faded {
    opacity: 0.3;
  }
  .highlighted {
    .opioid-value {
      display: block;
    }
  }
</style>
