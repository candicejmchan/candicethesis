<template>
  <div>
    <svg class="specialty-chart" ref="specialty"></svg>
    <div class="internal-medicine">
      Doctors working in
      <span class="selected-medicine dropdown-container">
        <b-nav-item-dropdown text="Internal Medicine" right>
            <b-dropdown-item href="#">EN</b-dropdown-item>
            <b-dropdown-item href="#">ES</b-dropdown-item>
            <b-dropdown-item href="#">RU</b-dropdown-item>
            <b-dropdown-item href="#">FA</b-dropdown-item>
        </b-nav-item-dropdown>
      </span>
      in
      <span class="selected-state dropdown-container">
        <b-nav-item-dropdown text="California" right>
            <b-dropdown-item href="#">EN</b-dropdown-item>
            <b-dropdown-item href="#">ES</b-dropdown-item>
            <b-dropdown-item href="#">RU</b-dropdown-item>
            <b-dropdown-item href="#">FA</b-dropdown-item>
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

// init chart constants
const WIDTH = 1200
const HEIGHT = 600
const MARGIN = {
  top: 20,
  bottom: 20,
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
const DATA_RANGE = 40

// chart vue code
export default {
  name: 'SpecialtyChart',
  data: () => {
    return {
      year: INITYEAR,
      data: DATA[INITYEAR],
      svgElement: '',
      svg: '',
      chartg: '',
      width: WIDTH - MARGIN.left - MARGIN.right,
      height: HEIGHT - MARGIN.top - MARGIN.bottom,
      yscale: '',
      xscale: ''
    }
  },
  mounted() {
      this.svgElement = this.$refs.specialty;
      this.initChart();
      // this.addSubHeading();
      this.drawChart();
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
    },
    addSubHeading: function() {
      // chart sub heading
      const subHeading = `
        Doctors working in <span class="selected-medicine">Internal Medicine</span>
        in <span class="selected-state">California</span> prescribe
        <span class="num-prescriptions">100000</span> opioid prescriptions
        per year
      `;

      d3.select('.internal-medicine')
        .html(subHeading)

      // this.svg.append('text')
      //     .classed('specialty-chart-sub-heading', true)
      //     .attr('x', headingWidth + 70)
      //     .attr('y', SUB_HEADING_SHIFT)
      //     .html(subHeading);
    },
    filterData: function() {
      // filter the data to get the relevant dataset
      return _.chain(this.data)
          // sort by the sum of opioid_prescriber_rate and la_opioid_prescriber_rate ascending
          .sortBy(d => (+d.opioid_prescriber_rate || 0) + (+d.la_opioid_prescriber_rate || 0))
          // make sure values are numbers
          .map(d => {
            _.keys(d).forEach(key => {
              d[key] = typeof +d[key] === 'undefined' || isNaN(+d[key]) ? d[key] : +d[key];
            })
            return d
          })
          // make it descending
          .reverse()
          // remove undefined / 0 values
          .filter(d => typeof d.la_opioid_prescriber_rate !== 'undefined' &&
                        d.la_opioid_prescriber_rate !== 0)
          // take the top values defined by DATA_RANGE
          .slice(0, DATA_RANGE)
          .value()
    },
    drawChart: function() {
      const data = this.filterData()

      const numFormat = d3.format('.0%');

      const gdata = this.chartg.selectAll('.ball-group').data(data)

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
        .attr('r', 5)
        .attr('cy', d => {
          const yval = this.yscale(d.opioid_prescriber_rate)
          console.log(yval)
          return yval;
        })
        .classed('opioid-prescriber-rate', true);

      // opioid_prescriber_rate text
      group.append('text')
        .attr('x', 10)
        .attr('y', d => this.yscale(d.opioid_prescriber_rate))
        .classed('opioid-prescriber-rate opioid-value', true)
        .text(d => numFormat(d.opioid_prescriber_rate/100))

      // opioid_prescriber_rate circle
      group.append('circle')
        .attr('r', 5)
        .attr('cy', d => this.yscale(d.la_opioid_prescriber_rate))
        .classed('la-opioid-prescriber-rate', true);

      // opioid_prescriber_rate text
      group.append('text')
        .attr('x', 10)
        .attr('y', d => this.yscale(d.la_opioid_prescriber_rate))
        .classed('la-opioid-prescriber-rate opioid-value', true)
        .text(d => numFormat(d.la_opioid_prescriber_rate/100))

      // white line joining the circles
      group.append('line')
          .attr('r', 5)
          .attr('y1', d => this.yscale(d.opioid_prescriber_rate))
          .attr('y2', d => this.yscale(d.la_opioid_prescriber_rate))
          .classed('opioid-prescriber-line', true);
    }
  }
}
</script>

<style lang="scss">
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
