<template lang="pug">
  .astro-event(v-if="nextEvent")
    span(v-bind:class="[nextEvent.type]") {{ nextEvent.type }}
    |
    | in
    |
    | {{ nextEventInString }}
</template>

<script>
  import moment from 'moment';
  import SunCalc from 'suncalc';
  import _ from 'underscore';

  let nextEvent = function(location) {
    var data = [];
    let now = moment();

    _.each([0, 1], function(i, e) {
      let date = moment(now).add(i, 'd').toDate(); // add() mutates original obj.
      let times = SunCalc.getTimes(date, location.lat, location.long);
      // this assumes sunrise is always before sunset in a given day 😂
      data.push({type: 'sunrise', time: moment(times.sunrise)});
      data.push({type: 'sunset', time: moment(times.sunset)});
    });

    data = _.select(data, function(e, i) {
      return e.time.isAfter(now);
    });

    return data[0];
  };

  let location = {
    name: 'Washington, DC',
    lat: 38.9047,
    long: -77.0315,
  };

  export default {
    data: function() {
      return {
        nextEvent: nextEvent(location),
        now: moment()
      };
    },
    mounted: function() {
      let component = this;

      setInterval(function ticker() {
        component.now = moment();
      }, 60 * 1000);
    },
    computed: {
      nextEventInString: function() {
        let then = moment(this.nextEvent.time);
        let hours = then.diff(this.now, 'hours');
        let minutes = then.diff(this.now, 'minutes') % 60;

        return `${hours}h ${minutes}m`;
      }
    },
  }
</script>

<style lang="scss" scoped>
  .astro-event {
    font-weight: 500;
  }

  .sunrise {
    color: orange;
  }
  .sunset {
    color: blue;
  }
</style>
