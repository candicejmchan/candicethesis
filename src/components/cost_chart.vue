<template>
  <div>
    <svg class="cost-chart" ref="cost"></svg>
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
      allocate
      <span class="cost-value">
        $50000
      </span>
      on opioids per year
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
const HEIGHT = 800
const MARGIN = {
  top: 20,
  bottom: 100,
  left: 100,
  right: 20
}
const SUB_HEADING_SHIFT = 300
const HEADING_SHIFT = 100
const headingWidth = WIDTH / 2 - 300
const heading = 'What is it Costing the United States?'
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
  name: 'CostChart',
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
      rscale_drug_cost: '',
      rscale_opioid_cost: '',
      yearScale: '',
      centerx: '',
      centery: '',
      randomRadii: _.range(10, 200, 10)
    }
  },
  mounted() {
      this.changeYearData();
      this.svgElement = this.$refs.cost;
      this.initChart();
      this.drawChart();
      this.addYearAxis();
  },
  methods: {
    initChart: function() {
      this.svg = d3.select(this.svgElement)
        .attr('width', WIDTH)
        .attr('height', HEIGHT)

      this.centerx = this.width / 2;
      this.centery = (this.height + SUB_HEADING_SHIFT)/2;

      const defs = this.svg.append('defs');
      const clip_total_cost = defs.append('clipPath')
        .attr('id', 'clip-path-total-cost');
      clip_total_cost.append('rect')
        .attr('x', 0)
        .attr('y', 0)
        .attr('width', this.centerx)
        .attr('height', this.centery + 200 + 10)

      const clip_opioid = defs.append('clipPath')
        .attr('id', 'clip-path-opioid');
      clip_opioid.append('rect')
        .attr('x', this.centerx)
        .attr('y', 0)
        .attr('width', this.centerx)
        .attr('height', this.centery + 200 + 10)

      // chart heading
      this.svg.append('text')
        .classed('specialty-chart-heading', true)
        .attr('x', headingWidth)
        .attr('y', HEADING_SHIFT)
        .text(heading);

      // y scale. shift height due to heading and sub heading
      this.rscale_drug_cost = d3.scaleLinear()
        .range([0, 200]);

      this.rscale_opioid_cost = d3.scaleLinear()
        .range([0, 200]);

      this.yearScale = d3.scalePoint()
        .range([0.2 * this.width, 0.8 * this.width])
        .domain(YEARS)
    },
    addYearAxis() {
      const axis = d3.axisBottom(this.yearScale);
      const g = this.svg.append('g')
        .attr('class', 'year-axis axis')
        .attr('transform', `translate(0, ${this.height + 50})`)
        .call(axis)

      g.append("text")
        .attr("y", 50)
        .attr("x", 0.5 * this.width)
        .attr('class', 'axis-label')
        .text("Year");

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

      this.internalMeds = chainData
          .map('specialty_description')
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
        this.selectedState = this.states.filter(d => d.state === 'CA')[0]
      if(this.selectedMed === '')
        this.selectedMed = this.internalMeds.filter(d => d === 'Internal Medicine')[0]
    },
    filterData: function() {
    },
    drawTotalCostCircles: function(){
      const total_all_states = _.chain(this.data)
        .filter(d => d.specialty_description === this.selectedMed || this.selectedMed === 'All')
        .map('total_drug_cost')
        .reduce( (sum, d) => d + sum)
        .value();

      const total_selected_state = _.chain(this.data)
        .filter(d => d.specialty_description === this.selectedMed || this.selectedMed === 'All')
        .filter(d => d.nppes_provider_state === this.selectedState.state || this.selectedState.state === 'ALL')
        .map('total_drug_cost')
        .reduce( (sum, d) => d + sum)
        .value();

      this.rscale_drug_cost.domain([0, total_all_states])

      const inBetState = _.range(100000, _.round(total_selected_state), 100000);
      const inBetAll = _.range(total_selected_state, _.round(total_all_states), 1000000);
      const inbetweenCircles = inBetState.concat(inBetAll);

      const group = this.svg.append('g').attr('class', 'total-cost-circles');

      const centerx = this.centerx;
      const centery = this.centery;

      const gdata = group.selectAll('circle')
                .data(inbetweenCircles);

      gdata.enter()
        .append('circle')
        .attr('class', 'circle-ring')
        .attr('clip-path', 'url(#clip-path-total-cost)')
        .attr('r', d => this.rscale_drug_cost(d))
        .classed('colored-circle', d => {
          if(d <= total_selected_state) {
            return true;
          } else {
            return false;
          }
        })
        .attr('cx', centerx)
        .attr('cy', centery)

      group.append('circle')
        .attr('class', 'total-all-circle')
        .attr('clip-path', 'url(#clip-path-total-cost)')
        .attr('r', d => this.rscale_drug_cost(total_all_states))
        .attr('cx', centerx)
        .attr('cy', centery)

      group.append('circle')
        .attr('class', 'total-sel-circle')
        .attr('clip-path', 'url(#clip-path-total-cost)')
        .attr('r', d => this.rscale_drug_cost(total_selected_state))
        .attr('cx', centerx)
        .attr('cy', centery)
    },
    drawOpioidCircles: function() {
      const total_all_states_opioid = _.chain(this.data)
        .filter(d => d.specialty_description === this.selectedMed || this.selectedMed === 'All')
        .map(d => {
          return (+d.opioid_drug_cost || 0) + (+d.la_opioid_drug_cost || 0)
        })
        .reduce( (sum, d) => d + sum)
        .value();

      const total_selected_state_opioid = _.chain(this.data)
        .filter(d => d.specialty_description === this.selectedMed || this.selectedMed === 'All')
        .filter(d => d.nppes_provider_state === this.selectedState.state || this.selectedState.state === 'ALL')
        .map(d => {
          return (+d.opioid_drug_cost || 0) + (+d.la_opioid_drug_cost || 0)
        })
        .reduce( (sum, d) => d + sum)
        .value();

      this.rscale_opioid_cost.domain([0, total_all_states_opioid])

      const inBetState = _.range(10000, _.round(total_selected_state_opioid), 10000);
      const inBetAll = _.range(total_selected_state_opioid, _.round(total_all_states_opioid), 100000);
      const inbetweenCircles = inBetState.concat(inBetAll);

      const centerx = this.centerx;
      const centery = this.centery;

      const group = this.svg.append('g').attr('class', 'opioid-circles');

      const gdata = group.selectAll('circle')
                .data(inbetweenCircles);

      gdata.enter()
        .append('circle')
        .attr('class', 'circle-ring')
        .attr('clip-path', 'url(#clip-path-opioid)')
        .attr('r', d => this.rscale_opioid_cost(d))
        .classed('colored-circle', d => {
          if(d <= total_selected_state_opioid) {
            return true;
          } else {
            return false;
          }
        })
        .attr('cx', centerx)
        .attr('cy', centery)

      group.append('circle')
        .attr('class', 'total-all-circle')
        .attr('clip-path', 'url(#clip-path-opioid)')
        .attr('r', d => this.rscale_opioid_cost(total_all_states_opioid))
        .attr('cx', centerx)
        .attr('cy', centery)

      group.append('circle')
        .attr('class', 'total-sel-circle')
        .attr('clip-path', 'url(#clip-path-opioid)')
        .attr('r', d => this.rscale_opioid_cost(total_selected_state_opioid))
        .attr('cx', centerx)
        .attr('cy', centery)
    },
    drawRandomCircles: function() {
      const group = this.svg.append('g').attr('class', 'random-circles');

      const gdata = group.selectAll('circle')
                .data(this.randomRadii);

      gdata.enter()
        .append('circle')
        .attr('class', 'circle-ring')
        .attr('r', d => d)
        .attr('cx', this.centerx)
        .attr('cy', this.centery)
    },
    drawChart: function() {
      // const data = this.filterData()
      const numFormat = d3.format('.0%');

      this.drawTotalCostCircles();
      this.drawOpioidCircles();

    }
  }
}
</script>

<style lang="scss">
  .axis {
    .tick {
      line { stroke: #fff; }
      text {
        fill: #fff;
        cursor: pointer;
        font-size: 12px;
      }
    }
    path {
      stroke: #fff;
    }
    .axis-label {
      fill: #fff;
      text-anchor: middle;
      font-size: 12px;
    }
  }
  .year-axis {
      .highlighted {
        font-size: 16px !important;
        fill: #F8E368 !important;
        font-weight: bold;
      }
  }
  .opioid-text {
    display: none;
    position: absolute;
    top: 200px;
    left: 25%;
    color: #fff;
    font-size: 20px;
    background-color: grey;
    .highlighted {
      color: #F8E368;
      font-weight: bold;
    }
  }
  .internal-medicine {
    position: absolute;
    top: 120px;
    left: 20%;
    color: #fff;
    font-size: 20px;
    .cost-value {
      color: #F8E368;
      font-weight: bold;
    }
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
  .visible {
    display: block;
  }
  .highlighted {
    .opioid-value {
      display: block;
    }
  }
  .mean-prescriptions {
    fill:#fff;
  }
  .total-all-circle {
    stroke: #fff;
    stroke-width: 3px;
    fill: none;
  }
  .total-sel-circle {
    stroke: #F8E368;
    stroke-width: 3px;
    fill: none;
  }
  .circle-ring {
    stroke: #fff;
    fill: none;
    stroke-width: 1px;
  }
  .colored-circle {
    stroke: #F8E368;
  }
</style>
