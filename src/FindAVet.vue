<template lang="pug">
div
  .controls
    label Radius
      select(name="rad" v-model="range")
        option(v-for="(r,i) in ranges" v-bind:value="r" v-bind:selected="i == 0") {{r}} {{distanceUnits}}

    label Zip Code
      input(name="origin" id="origin" v-model="origin")
    
    button(@click="geocode") Search
    
  .results
    table(width="100%" v-if="searched")
      thead
        tr
          th Practice
          th Address
          th Distance
      tbody
        template(v-if="practices.length > 0" )
          tr(v-for="p in practices")
            td {{p.custom_s_name}}
            td
              | {{p.custom_s_address1}} <br/>

              | {{p.custom_s_city}},{{p.custom_s_state}} <br/>
              | {{p.custom_s_postal_code}}
            td {{ (p._dist_/distanceConversionFactor).toFixed(2) }} {{distanceUnits}}
        template(v-else)
          tr
            td(colspan="3") No results found
</template>

<script>
export default {
  name: "find-a-vet",
  props: {
    collection: { type: String, required: true},
    products: { type: Array, required: true },
    usingMiles: { type: Boolean, default: true },
    ranges: { type: Array, default: function() { return [5, 10, 15, 20, 25] }},
    defaultRange: { type:Number, default: 5 }
  },
  created: function() {
    this.cps = new CrownPeakSearch(this.collection);
    this.range = this.defaultRange;
  },    
  data () {
    return {
        origin: '',
        range: 0,
        practices: [],
        searched: false,
        geo: {
          field: "custom_p_location", // could be omitted - this is the default
          outputfield: "_dist_", // could be omitted - this is the default
        }
    }
  },
  computed: {

    distanceUnits: function() {
      return this.usingMiles ? "miles" : "km";
    },
    distanceConversionFactor: function() {
      return this.usingMiles ? 1.609344 : 1;
    },
    productsFilter: function() {
      var queries = [];
      queries.push("custom_s_type:veterinary_practice_data");
      return this.products.map(function (i) {
        queries.push("custom_ss_products:"+i);
      })
    }
  },
  methods: {
    geocode: function() {
      var vm = this;
      var origin = this.origin;
      if (origin !== '') {
        var xhr=new XMLHttpRequest();
        xhr.onload = function() {
          if (xhr.status == 200) {
            var location = xhr.response.results[0].geometry.location;
            vm.geo.location = { lat: location.lat, lon: location.lng };
            vm.geo.range = { distance: vm.range, units: vm.distanceUnits }
            console.log(location);
            vm.cps.geo(vm.geo);
            vm.cps.query("*:*", 1, vm.productsFilter).done(function (data) { vm.searched = true; vm.practices = data.response.docs; })
          } 
          else {
            console.log("Something went wrong");
          }
        };
        xhr.open('GET', 'https://maps.googleapis.com/maps/api/geocode/json?address=' + origin + '&key=AIzaSyDZcYFJhwlZxfl4O7N4C0wSTG9D4IFQHJc');
        xhr.responseType = 'json';
        xhr.send();
      }
    }
  }
}
</script>

<style scoped>
table {
  border-collapse: collapse;
  border-bottom: 2px solid silver;
}
tr {
  vertical-align: top;
}

thead tr {
  background: silver;
}
tbody tr:nth-child(odd) {
  background: #eee;
}
th, td {
  padding: 0.5em;
}
</style>