<template>
  <div class="constraints-graph">
    <h1>Constraint Analysis</h1>
    <div id="constraints-graph-ui">
      <div id="chart" v-if="isOnline">
        <GChart type="ComboChart" :data="chartData" :options="chartOptions"/>
        <label>Ymin: <input v-model.number="ymin"/></label><br>
        <label>Ymax: <input v-model.number="ymax"/></label>
      </div>
      <div v-else>Charts are not available offline</div>
      <div id="inputs">
        <form id="constraint-form">
          <select v-model="editWhichConstraint">
            <option disabled value="">Select a constraint to edit</option>
            <option value="editTakeoffRun">Takeoff run</option>
            <option value="editSustainedTurn">Sustained turn</option>
            <option value="editClimbRate">Climb rate</option>
            <option value="editClimbAngle">Climb angle</option>
          </select>
          <fieldset v-show="editWhichConstraint === 'editTakeoffRun'">
            <legend>Takeoff run</legend>
            <Equation data="$$\frac{T_{TO}}{W_{TO}} = \frac{\beta^2}{\alpha} \cdot
              \frac{k_{TO}^2}{s_G\cdot\rho_{TO} \cdot g_0\cdot C_{L_{TO}}}\cdot
              \frac{W_{TO}}{S}$$"/>
            <label><Equation data="$\beta$" title="Weight fraction"/>
              <input type="range" min="0" max="1" step=".05"
                v-model.number="takeoffRunConstraint.weightFraction"/>
              <input v-model.number="takeoffRunConstraint.weightFraction"/>
            </label>
            <label><Equation data="$\alpha$" title="Thrust fraction"/>
              <input type="range" min="0" max="1" step=".05"
                v-model.number="takeoffRunConstraint.thrustFraction"/>
              <input v-model.number="takeoffRunConstraint.thrustFraction"/>
            </label>
            <label><Equation data="$k_{TO}$" title="Takeoff speed coefficient"/>
              <input type="range" min="1" max="2" step="0.05"
                v-model.number="takeoffRunConstraint.takeoffSpeedCoeff"/>
              <input v-model.number="takeoffRunConstraint.takeoffSpeedCoeff"/>
            </label>
            <label><Equation data="$C_{L_{TO}}$" title="Takeoff lift coefficient"/>
              <input type="range" min="0" max="3" step="0.05"
                v-model.number="takeoffRunConstraint.liftCoefficientTakeoff"/>
              <input v-model.number="takeoffRunConstraint.liftCoefficientTakeoff"/>
            </label>
            <label><Equation data="$s_{G}$" title="Ground run"/>
              <input type="range" min="0" max="3000" step="100"
                v-model.number="takeoffRunConstraint.groundRun"/>
              <span><input v-model.number="takeoffRunConstraint.groundRun"/>
              <Equation data="$ft$"/></span>
            </label>
            <label><Equation data="$\rho_{TO}$" title="Density at takeoff"/>
              <input type="range" min="0.0004" max=".003" step="0.0001"
                v-model.number="takeoffRunConstraint.densityAtTakeoff"/>
              <span><input v-model.number="takeoffRunConstraint.densityAtTakeoff"/>
              <Equation data="$slugs/ft^3$"/></span>
            </label>
          </fieldset>
          <fieldset v-show="editWhichConstraint === 'editSustainedTurn'">
            <legend>Sustained turn</legend>
            <Equation data="$$\frac{T_{TO}}{W_{TO}} = q\left[\frac{C_{D_{min}}}{(W_{TO}/S)}\right]
              + k\left(\frac{n}{q}\right)^2\left(\frac{W_{TO}}{S}\right)$$"/>
            <label><Equation data="$q$" title="Dynamic pressure"/>
              <input type="range" min="0" max="1000" step=".1"
                v-model.number="sustainedTurnConstraint.dynamicPressure"/>
              <span><input v-model.number="sustainedTurnConstraint.dynamicPressure"/>
              <Equation data="$lb/ft^2$"/></span>
            </label>
            <label><Equation data="$C_{D_{min}}$" title="Minimum drag coefficient"/>
              <input type="range" min="0" max="0.05" step=".005"
                v-model.number="sustainedTurnConstraint.minDragCoeff"/>
              <input v-model.number="sustainedTurnConstraint.minDragCoeff"/>
            </label>
            <label><Equation data="$k$" title="Unknown Coefficient"/>
              <input type="range" min="0" max="2" step="0.05"
                v-model.number="sustainedTurnConstraint.unknownCoeff"/>
              <input v-model.number="sustainedTurnConstraint.unknownCoeff"/>
            </label>
            <label><Equation data="$n$" title="Load factor"/>
              <input type="range" min="0" max="2" step="0.05"
                v-model.number="sustainedTurnConstraint.loadFactor"/>
              <input v-model.number="sustainedTurnConstraint.loadFactor"/>
            </label>
          </fieldset>
          <fieldset v-show="editWhichConstraint === 'editClimbRate'">
            <legend>Climb rate</legend>
            <Equation data="$$\frac{T_{TO}}{W_{TO}} = \frac{V_v}{V_{cruise}}+ \frac{q}{W_{TO}/S}
              \cdot C_{D_{min}} + \frac{C_{D_{induced}}}{q}\cdot\frac{W_{TO}}{S}$$"/>
            <label><Equation data="$q$" title="Dynamic pressure"/>
              <input type="range" min="0" max="1000" step=".1"
                v-model.number="climbRateConstraint.dynamicPressure"/>
              <span><input v-model.number="climbRateConstraint.dynamicPressure"/>
              <Equation data="$lb/ft^2$"/></span>
            </label>
            <label><Equation data="$V_{v}$" title="Vertical speed"/>
              <input type="range" min="0" max="40" step="1"
                v-model.number="climbRateConstraint.verticalSpeed"/>
              <span><input v-model.number="climbRateConstraint.verticalSpeed"/>
              <Equation data="$ft/s$"/></span>
            </label>
            <label><Equation data="$V_{cruise}$" title="Cruise speed"/>
              <input type="range" min="0" max="500" step="10"
                v-model.number="climbRateConstraint.cruiseSpeed"/>
              <span><input v-model.number="climbRateConstraint.cruiseSpeed"/>
              <Equation data="$ft/s$"/></span>
            </label>
            <label><Equation data="$C_{D_{min}}$" title="Minimum drag coefficient"/>
              <input type="range" min="0" max="0.05" step=".005"
                v-model.number="climbRateConstraint.minDragCoeff"/>
              <input v-model.number="climbRateConstraint.minDragCoeff"/>
            </label>
            <label><Equation data="$C_{D_{induced}}$" title="Lift induced drag coefficient"/>
              <input type="range" min="0" max="0.05" step=".005"
                v-model.number="climbRateConstraint.liftInducedDragCoeff "/>
              <input v-model.number="climbRateConstraint.liftInducedDragCoeff"/>
            </label>
          </fieldset>
          <fieldset v-show="editWhichConstraint === 'editClimbAngle'">
            <legend>Climb angle</legend>
            <Equation data="$$\frac{T_{TO}}{W_{TO}} = \frac{\beta}{\alpha} \left[ k\left(
              \frac{\beta W_{TO}}{qS}\right) + \frac{C_{D_{min}}}{\left(\frac{\beta W_{TO}}{qS}
              \right)} + \sin\theta\right]$$"/>
            <label><Equation data="$\beta$" title="Weight fraction"/>
              <input type="range" min="0" max="1" step=".05"
                v-model.number="climbAngleConstraint.weightFraction"/>
              <input v-model.number="climbAngleConstraint.weightFraction"/>
            </label>
            <label><Equation data="$\alpha$" title="Thrust fraction"/>
              <input type="range" min="0" max="1" step=".05"
                v-model.number="climbAngleConstraint.thrustFraction"/>
              <input v-model.number="climbAngleConstraint.thrustFraction"/>
            </label>
            <label><Equation data="$k$" title="Lift induced drag constant"/>
              <input type="range" min="0" max="10" step=".1"
                v-model.number="climbAngleConstraint.liftInducedDragConst"/>
              <input v-model.number="climbAngleConstraint.liftInducedDragConst"/>
            </label>
            <label><Equation data="$q$" title="Dynamic pressure"/>
              <input type="range" min="0" max="1000" step="1"
                v-model.number="climbAngleConstraint.dynamicPressure"/>
              <span><input v-model.number="climbAngleConstraint.dynamicPressure"/>
              <Equation data="$lb/ft^2$"/></span>
            </label>
            <label><Equation data="$C_{D_{min}}$" title="Minimum drag coefficient"/>
              <input type="range" min="0" max="0.05" step=".005"
                v-model.number="climbAngleConstraint.minDragCoeff"/>
              <input v-model.number="climbAngleConstraint.minDragCoeff"/>
            </label>
            <label><Equation data="$\theta$" title="Climb angle"/>
              <input type="range" min="0" max="20" step=".1"
                v-model.number="climbAngleConstraint.climbAngle "/>
              <span><input v-model.number="climbAngleConstraint.climbAngle"/>
              <Equation data="$\deg$"/></span>
            </label>
          </fieldset>
        </form>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';
import { GChart, loadGoogleCharts } from 'vue-google-charts';
import Equation from '@/components/Equation.vue';
import {
  TakeoffRunConstraint,
  SustainedTurnConstraint,
  ClimbRateConstraint,
  ClimbAngleConstraint,
} from '@/interfaces/constraints';

@Component({
  components: {
  GChart,
  Equation
  }
  })
export default class ConstraintsGraph extends Vue {
  /**
   * whether the app is online or not
   */
  /* eslint-disable-next-line class-methods-use-this */
  get isOnline() {
    return window.navigator.onLine;
  }

  /**
   * data fields for take off run constraint
   */
  public takeoffRunConstraint: TakeoffRunConstraint = {
    weightFraction: 1,
    thrustFraction: 1,
    takeoffSpeedCoeff: 1.2,
    liftCoefficientTakeoff: 2.9,
    groundRun: 1200, // ft
    densityAtTakeoff: 0.0023768924, // slugs/ft^3
  }

  /**
   * data fields for sustained turn constraint
   */
  public sustainedTurnConstraint: SustainedTurnConstraint = {
    dynamicPressure: 59, // lb/ft^2
    minDragCoeff: 0.03,
    loadFactor: 1.41,
    unknownCoeff: 0.045,
  }

  /**
   * data fields for sustained turn constraint
   */
  public climbRateConstraint: ClimbRateConstraint = {
    dynamicPressure: 59, // lb/ft^2
    minDragCoeff: 0.03,
    liftInducedDragCoeff: 0.022,
    verticalSpeed: 20, // ft/s
    cruiseSpeed: 220, // ft/s
  }

  /**
   * data fields for climb angle constraint
   */
  public climbAngleConstraint: ClimbAngleConstraint = {
    weightFraction: 1,
    thrustFraction: 1,
    liftInducedDragConst: 0.022,
    dynamicPressure: 59,
    minDragCoeff: 0.03,
    climbAngle: 3.1,
  }

  /**
   * which constraint fields to show for editing
   */
  public editWhichConstraint: string = '';

  /**
   * the values for the x-axis
   */
  private xAxis = Array.from({ length: 500 }, (v, k) => (k+1) / 10);

  /**
   * an array of constraint functions that calculate the thrust to weight ratio
   * and take the wing loading (W_TO)/S as an input paramater
   */
  private constraintFunctions = [
    this.takeoffRun,
    this.sustainedTurn,
    this.climbRate,
    this.climbAngle,
  ]

  /**
   * the minimum thrust to weight ratio
   */
  get minThrustToWeight(): number {
    let minThrustToWeight: number = this.constraintFunctions.reduce(
      (max, constraint) =>
        (max > constraint.apply(this, [this.xAxis[0]]) ? max
          : constraint.apply(this, [this.xAxis[0]]))
      , 0,
    );
    let wingLoadingAtMinThrustToWeight: number = this.xAxis[0];
    this.xAxis.forEach((wingLoading) => {
      const minThrustToWeightAtThisWingLoading = this.constraintFunctions.reduce(
        (max, constraint) =>
          (max > constraint.apply(this, [wingLoading]) ? max
            : constraint.apply(this, [wingLoading]))
        , 0,
      );
      if (minThrustToWeightAtThisWingLoading < minThrustToWeight) {
        minThrustToWeight = minThrustToWeightAtThisWingLoading;
        wingLoadingAtMinThrustToWeight = wingLoading;
      }
      minThrustToWeight = Math.min(minThrustToWeight, minThrustToWeightAtThisWingLoading);
    });
    return minThrustToWeight;
  }

  /**
   * data for the google chart
   * @returns {(string | number | null | object)[][]} a two-dimensional array of
   * data to be processed with google charts arrayToDataTable()
   */
  get chartData(): (string | number | null | object)[][] {
    const google = loadGoogleCharts('current', { packages: ['corechart'] });
    const header = [
      { label: 'Wing loading', type: 'number' }, { label: 'Takeoff run', type: 'number' },
      { label: 'Sustained turn', type: 'number' }, { label: 'Climb rate', type: 'number' },
      { label: 'Climb angle', type: 'number' },
      // separate series for optimum point annotation
      { type: 'number' }, { type: 'string', role: 'annotation' },
      { type: 'string', role: 'annotationText', p: { html: true } },
    ];
    const rows = this.xAxis.map((wingLoading) => {
      const thrustToWeightRatios = this.constraintFunctions.map(constraint =>
        constraint.apply(this, [wingLoading]));
      let optimumPoint = null;
      let annotation = null;
      let annotationText = null;
      if (Math.max(...thrustToWeightRatios) === this.minThrustToWeight) {
        optimumPoint = this.minThrustToWeight;
        annotation = 'Optimum';
        annotationText = `Optimum T/W: ${this.minThrustToWeight}<br>Optimum W/S: ${wingLoading}`;
      }
      return [
        wingLoading,
        ...thrustToWeightRatios,
        optimumPoint,
        annotation,
        annotationText];
    });
    const data = [header, ...rows];
    return data;
  }

  /**
   * the max value on the vertical axis
   */
  private ymax:number = 0.5;

  /**
   * the min value on the vertical axis
   */
  private ymin:number = 0;

  /**
   * options to use for the google chart
   */
  private get chartOptions() {
    return {
      title: 'Design space',
      height: 500,
      tooltip: { isHtml: true },
      seriesType: 'area',
      series: {
        4: { visibleInLegend: false },
      },
      hAxis: {
        title: 'W/S',
      },
      vAxis: {
        title: 'T/W',
        viewWindow: {
          max: this.ymax,
          min: this.ymin,
        },
      },
      animation: {
        startup: true,
        duration: 300,
        easing: 'out',
      },
    };
  }

  /**
   * Thrust to weight ratio for take off run as a function of wing loading
   * @param {number} wingLoading the wing loading (W_TO)/S [lbf/ft^2]
   * @returns {number} thrust to weight ratio
   */
  public takeoffRun(wingLoading: number): number {
    const b = this.takeoffRunConstraint.weightFraction;
    const a = this.takeoffRunConstraint.thrustFraction;
    const k = this.takeoffRunConstraint.takeoffSpeedCoeff;
    const Sg = this.takeoffRunConstraint.groundRun;
    const rho = this.takeoffRunConstraint.densityAtTakeoff;
    const CL = this.takeoffRunConstraint.liftCoefficientTakeoff;
    return ((b**2) / a) * ((k**2) / (Sg * rho * 32.2 * CL)) * wingLoading;
  }

  /**
   * Thrust to weight ratio for landing distance as a function of wing loading
   * @param {number} wingLoading the wing loading (W_TO)/S
   * @returns {number} thrust to weight ratio
   */
  public sustainedTurn(wingLoading: number): number {
    const q = this.sustainedTurnConstraint.dynamicPressure;
    const n = this.sustainedTurnConstraint.loadFactor;
    const CDmin = this.sustainedTurnConstraint.minDragCoeff;
    const k = this.sustainedTurnConstraint.unknownCoeff;
    return q * ((CDmin / wingLoading) + (k * ((n / q)**2) * wingLoading));
  }

  /**
   * Thrust to weight ratio for climb rate as a function of wing loading
   * @param {number} wingLoading the wing loading (W_TO)/S
   * @returns {number} thrust to weight ratio
   */
  public climbRate(wingLoading: number): number {
    const q = this.climbRateConstraint.dynamicPressure;
    const Vv = this.climbRateConstraint.verticalSpeed;
    const V = this.climbRateConstraint.cruiseSpeed;
    const CDmin = this.climbRateConstraint.minDragCoeff;
    const CDinduced = this.climbRateConstraint.liftInducedDragCoeff;
    return (Vv / V) + ((q / wingLoading) * CDmin) + ((CDinduced / q) * wingLoading);
  }

  /**
   * Thrust to weight ratio for climb angle as a function of wing loading
   * @param {number} wingLoading the wing loading (W_TO)/S
   * @returns {number} thrust to weight ratio
   */
  public climbAngle(wingLoading: number): number {
    const b = this.climbAngleConstraint.weightFraction;
    const a = this.climbAngleConstraint.thrustFraction;
    const k = this.climbAngleConstraint.liftInducedDragConst;
    const q = this.climbAngleConstraint.dynamicPressure;
    const CDmin = this.climbAngleConstraint.minDragCoeff;
    const theta = this.climbAngleConstraint.climbAngle * (Math.PI / 180);
    return (b / a) * ((k * (b / q) * wingLoading) + (CDmin / ((b / q) * wingLoading))
      + Math.sin(theta));
  }
}
</script>

<style lang="scss" scoped>
  $desktopWidth: 1325px;
  #constraints-graph-ui {
    display: flex;
    flex-wrap: wrap;
  }

  #chart {
    width:100%;
  }

  #inputs {
    flex: 1;
  }
  @media screen and (min-width: $desktopWidth) {
    #chart {
      flex: 1;
      max-width: 70%;
    }

    #inputs {
      flex: 1;
      max-width:30%;
      order: -1;
    }
  }
  #constraint-form {
    display: flex;
    flex-direction: column;

    select {
      flex: 1;
      max-width: 300px;
      margin: 1.5em auto;
    }

    fieldset {
      padding: 0.5em;
      flex: 1;
      max-width: 600px;
      margin: 0 auto;
    }

    label {
      display: flex;
      flex-direction: column;
      padding: 0.5em 0;

      & > input[type="range"] {
        width: 100%;
        margin: 0.5em 0;
        padding: 0;
      }

      & > span:last-child {
        display: flex;

        & > input {
          flex: 1;
        }

        & > span {
          margin-left: 0.5em;
        }
      }
    }
  }
</style>