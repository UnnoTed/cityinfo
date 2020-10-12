<template>
  <div>
    <!-- <div class="q-pa-md row items-start q-gutter-sm full-width"> -->
    <q-drawer show-if-above :width="300" elevated>
      <q-scroll-area class="fit">
        <q-list bordered class="rounded-borders" style="max-width: 350px">
          <q-item-label header>Where {{ where }}</q-item-label>

          <q-item>
            <q-option-group
              v-model="where"
              :options="where_options"
              color="primary"
              inline
              type="checkbox"
            />
          </q-item>
        </q-list>
        <q-list bordered class="rounded-borders" style="max-width: 350px">
          <q-item-label header>How {{ how }}</q-item-label>

          <q-item>
            <q-input
              filled
              type="number"
              v-model="filters.safe.level"
              label="Level"
              lazy-rules
              :rules="[
                (val) =>
                  (val > 0 && val < 5) ||
                  'Should be greater than 0 and less than 5',
              ]"
            >
              <template v-slot:before>
                <q-checkbox
                  v-model="filters.safe.enabled"
                  label="Safe"
                  color="primary"
                  inline
                  type="checkbox"
                />
              </template>
            </q-input>
          </q-item>

          <q-item>
            <q-option-group
              v-model="how"
              :options="how_options"
              color="primary"
              inline
              type="checkbox"
            />
          </q-item>
        </q-list>

        <q-list bordered class="rounded-borders" style="max-width: 350px">
          <q-item-label header>Cost</q-item-label>

          <q-item>
            <q-checkbox
              v-model="filters.cost.enabled"
              label=""
              color="primary"
              inline
              type="checkbox"
            />
          </q-item>
          <q-item>
            <q-select
              filled
              v-model="filters.cost.mode.model"
              :options="filters.cost.mode.options"
              label="Mode"
            />
          </q-item>
          <q-item>
            <q-input
              filled
              type="number"
              v-model="filters.cost.cost"
              label="Local"
              lazy-rules
              :rules="[
                (val) =>
                  (val > 0 && val < 999999999) ||
                  'Should be greater than 0 and less than 999999999',
              ]"
            >
            </q-input>
          </q-item>
        </q-list>

        <q-list bordered class="rounded-borders" style="max-width: 350px">
          <q-item-label header>Country</q-item-label>
          <q-item>
            <!-- <q-input filled v-model="filters.country" label="Name"> </q-input> -->

            <q-select
              filled
              v-model="filters.country"
              use-input
              input-debounce="0"
              label="Name"
              :options="countries"
              @filter="filterCountry"
            >
              <template v-slot:no-option>
                <q-item>
                  <q-item-section class="text-grey">
                    No results
                  </q-item-section>
                </q-item>
              </template>
            </q-select>
          </q-item>
        </q-list>

        <q-list bordered class="rounded-borders" style="max-width: 350px">
          <q-item-label header>Health</q-item-label>

          <q-item>
            <q-option-group
              v-model="health"
              :options="health_options"
              color="primary"
              inline
              type="checkbox"
            />
          </q-item>
        </q-list>
        <q-list bordered class="rounded-borders" style="max-width: 350px">
          <q-item-label header>Size</q-item-label>

          <q-item>
            <q-option-group
              v-model="size"
              :options="size_options"
              color="primary"
              inline
              type="checkbox"
            />
          </q-item>
        </q-list>
        <q-list bordered class="rounded-borders" style="max-width: 350px">
          <q-item-label header>Climate</q-item-label>

          <q-item>
            <q-option-group
              v-model="climate"
              :options="climate_options"
              color="primary"
              inline
              type="checkbox"
            />
          </q-item>
        </q-list>
      </q-scroll-area>
    </q-drawer>

    <q-drawer
      side="right"
      show-if-above
      :width="800"
      bordered
      v-if="selected && selected.name !== '' && selected.things"
    >
      <!-- <q-scroll-area class="fit">  -->

      <div class="text-h5">
        {{ selected.name }}
      </div>
      <div class="text-caption">
        {{ selected.country }}
      </div>

      <q-list
        bordered
        separator
        class="row"
        v-for="section in ['ranking', 'guide', 'cost']"
        :key="'sec-' + section"
      >
        <q-item
          class="col-3"
          clickable
          v-ripple
          v-for="(thing, id) in selectedThings.filter(
            (t) => t.v.section === section
          )"
          :key="'thing-' + selected.short_slug + '-' + id"
        >
          <q-item-section
            v-if="thing.k !== 'proscons'"
            :class="'section-' + thing.v.section"
          >
            <q-item-label>{{ things[thing.k] }}</q-item-label>
            <q-item-label caption>{{ thing.v.label }}</q-item-label>
          </q-item-section>
        </q-item>
      </q-list>

      <q-list bordered separator class="row">
        <q-item
          class="col-3"
          clickable
          v-ripple
          v-for="(pc, idx) in selectedThings.filter(
            (t) => t.k === 'proscons'
          )[0].v"
          :key="'pc-' + idx"
        >
          <q-item-section>
            <q-item-label>{{ pc }}</q-item-label>
          </q-item-section>
        </q-item>
      </q-list>

      <!-- </q-scroll-area> -->
    </q-drawer>

    <q-page>
      <q-table
        :grid="true"
        class="cities"
        title="Cities"
        :data="cfiltered"
        :columns="columns"
        row-key="index"
        virtual-scroll
        :virtual-scroll-item-size="48"
        :virtual-scroll-sticky-size-start="48"
        :pagination="pagination"
        :rows-per-page-options="[0]"
      >
        <template v-slot:item="props">
          <div class="q-pa-xs col-xs-12 col-sm-12 col-md-12 col-lg-6 col-xl-3">
            <q-card class="city" flat bordered>
              <q-img
                :src="imageURL(props.row.image)"
                @mouseenter="
                  show_things[props.row.short_slug] = !show_things[
                    props.row.short_slug
                  ];
                  selected = props.row;
                "
              >
                <div class="absolute-center">
                  <div class="text-h5">{{ props.row.name }}</div>
                  <div class="text-caption text-center">
                    {{ props.row.country }}
                  </div>
                </div>
                <div class="absolute-bottom">
                  <div>
                    <div>${{ props.row.cost_for_local_in_usd }} / mo</div>
                    <div>
                      <q-linear-progress
                        size="20px"
                        :color="safetyColor(props.row.safety_level)"
                        :value="(props.row.safety_level * 20) / 100"
                        label="Safety"
                        class="q-mt-md"
                      />
                    </div>
                  </div>
                </div>
              </q-img>
            </q-card>
          </div>
        </template>
      </q-table>
    </q-page>
  </div>
</template>

<script>
import data from "../data.json";

var show_things = {};
var countries = [];
data.cities.forEach((c) => {
  show_things[c.short_slug] = false;
  if (countries.indexOf(c.country) === -1) {
    countries.push(c.country);
  }
});

export default {
  name: "PageIndex",
  computed: {
    cfiltered() {
      // return this.data;
      return this.data.cities.filter((c) => {
        // console.log(this.where);
        if (
          (this.where.indexOf("eu") !== -1 && c.region !== "Europe") ||
          (this.where.indexOf("usa") !== -1 && c.country_code !== "US") ||
          (this.where.indexOf("na") !== -1 && c.region !== "North America") ||
          (this.where.indexOf("la") !== -1 && c.region !== "Latin America") ||
          (this.where.indexOf("euu") !== -1 && c.region !== "Europe") ||
          (this.where.indexOf("as") !== -1 && c.region !== "Asia") ||
          (this.where.indexOf("me") !== -1 && c.region !== "Middle East") ||
          (this.where.indexOf("oc") !== -1 && c.region !== "Oceania") ||
          (this.where.indexOf("sp") !== -1 && c.region !== "Space") ||
          (this.where.indexOf("af") !== -1 && c.region !== "Africa") ||
          (this.where.indexOf("nis") !== -1 &&
            (c.country !== "Bulgaria" ||
              c.country !== "Romania" ||
              c.country !== "Croatia" ||
              c.country !== "Cyprus" ||
              c.country !== "Ireland" ||
              c.country !== "United Kingdom" ||
              c.country !== "Albania" ||
              c.country !== "Armenia" ||
              c.country !== "Azerbaijan" ||
              c.country !== "Belarus" ||
              c.country !== "Bosnia" ||
              c.country !== "Herzegovina" ||
              c.country !== "Macedonia" ||
              c.country !== "Moldova" ||
              c.country !== "Montenegro" ||
              c.country !== "Serbia" ||
              c.country !== "Ukraine"))
        ) {
          return false;
        }

        if (
          this.filters.safe.enabled &&
          c.safety_level < Number(this.filters.safe.level)
        ) {
          return false;
        }

        if (
          this.filters.cost.enabled &&
          ((this.filters.cost.mode.model.value === "lt" &&
            c.cost_for_local_in_usd > Number(this.filters.cost.cost)) ||
            (this.filters.cost.mode.model.value === "gt" &&
              c.cost_for_local_in_usd < Number(this.filters.cost.cost)))
        ) {
          return false;
        }

        if (this.filters.country !== "" && c.country !== this.filters.country) {
          return false;
        }

        return true;
      });
    },
    selectedThings() {
      var s = [];

      for (var i in this.selected.things) {
        s.push({
          k: i,
          v: this.selected.things[i],
        });
      }

      return s;
    },
  },
  data() {
    return {
      data: Object.freeze(data),
      filtered: this.cfiltered,
      pagination: {
        page: 1,
        rowsPerPage: 12,
      },
      show_things,
      countries,
      selected: {},
      filters: {
        safe: {
          enabled: false,
          level: 3.85,
        },
        cost: {
          enabled: false,
          cost: 1000,
          mode: {
            model: { label: "Less than", value: "lt" },
            options: [
              { label: "Greater than", value: "gt" },
              { label: "Less than", value: "lt" },
            ],
          },
        },
        country: "",
      },
      things: {
        score: "	⭐️ Overall Score",
        quality: "👍 Quality of life score",
        family: "👶 Family score",
        cost: "💵 Cost",
        internet: "📡 Internet",
        fun: "😝 Fun",
        temperature: "⛅️ Temperature (now)",
        humidity: "💦 Humidity (now)",
        air_quality: "💨 Air quality (now)",
        safety: "👌 Safety",
        education: "🎓 Education level",
        liked: "❤️ Liked by members",
        english: "🙊 English speaking",
        density: "😤 People density",
        walkability: "🚶 Walkability",
        peace: "✌️ Peace",
        traffic_safety: "🚦 Traffic safety",
        hospitals: "🏥 Hospitals",
        happiness: "😄 Happiness",
        nightlife: "🍸 Nightlife",
        free_wifi: "📶 Free WiFi in city",
        places_to_work: "🖥 Places to work from",
        ac: "❄️ A/C or heating",
        friendly: "😁 Friendly to foreigners",
        freedom: "🗯 Freedom of speech",
        racial_tolerance: "🤚🏿🤚🏻 Racial tolerance",
        female_friendly: "👩 Female friendly",
        lgbt_friendly: "🌈 LGBTQ+ friendly",
        startup: "🎅 Startup Score",
        region: "🌍 Region",
        country: "🚩 Country",
        average_trip_duration: "⏱ Average trip duration",
        internet_speed: "📡 Internet speed (avg)",
        weather: "⛅️ Weather (now)",
        power: "🔌 Power",
        best_neighborhood: "🧔 Best neighborhood to stay",
        upcoming_neighborhood: "🚀 Upcoming neighborhood",
        best_taxi: "🚕 Best taxi app (in country)",
        travel_medical_insurance: "🚑 Travel medical insurance",
        best_wifi: "📱 Best wireless carrier",
        atm: "🏧 Suggested ATM take out:",
        tipping: "💸 Tipping",
        cashless: "💳 Cashless society",
        best_cowork: "💻 Best coworking space",
        safe_tap_water: "🚰 Safe tap water",
        return_rate: "♻️ Return rate",
        visitors: "📸 Visitors per year",
        tourists: "📸 Tourists now",
        population: "👨‍👩‍👧‍👦 Population",
        population_density: "😤 Population density",
        foreign_land: "🏞 Foreign land ownership allowed",
        gender_ratio: "👫 Gender ratio (overall)",
        gender_ratio_young: "👫 Gender ratio (young adults)",
        religious: "⛪️ Religious government",
        online_electronics_shop: "💻 Online electronics shop",
        apartments: "🏠 Apartment listings",
        best_short_air: "✈️ Best short-haul air carrier",
        best_air: "✈️ Best int'l air carrier",
        cost_nomad: "💵 Cost of living for nomad",
        cost_expat: "💵 Cost of living for expat",
        cost_family: "💵 Cost of living for family",
        cost_local: "💵 Cost of living for local",
        studio_rent: "🏠 1br studio rent in center",
        coworking: "🏢 Coworking",
        hotel: "🏨 Hotel",
        airbnb: "🏠 Airbnb",
        dinner: "🍛 Dinner",
        coca_cola: "🥤 Coca-Cola (0.3L)",
        beer: "🍺 Beer (0.5L)",
        coffee: "☕️ Coffee",
        estimated_tax_50_000: "💰 Estimated tax on $50,000",
        estimated_tax_100_000: "💰 Estimated tax on $100,000",
        estimated_tax_250_000: "💰 Estimated tax on $250,000",
        healthcare: "🏥 Healthcare",
        idr_usd: "💸 100,000 IDR in USD",
        best_alt_coworking: "💻 Best alt. coworking space",
        best_coffee: "☕️ Best coffee place",
        best_alt_coffee: "☕️ Best alt. coffee place",
        best_hospital: "🏥 Best hospital",
        air_quality_annual: "💨 Air quality (annual)",
        air_quality_avg: "💨 Air quality (annual avg)",
        best_coffee_247: "🏪 Best 24/7 coffee place",
        sales_tax: " Sales tax (VAT/GST)",
        individual_tax_min: " Individual tax rate (min)",
        individual_tax_max: " Individual tax rate (max)",
        corporate_tax: "💰 Corporate tax rate",
      },
      columns: [
        {
          name: "name",
          label: "Name",
          field: "name",
        },
        {
          name: "name",
          required: true,
          label: "Dessert (100g serving)",
          align: "left",
          field: (row) => row.name,
          format: (val) => `${val}`,
          sortable: true,
        },
        {
          name: "calories",
          align: "center",
          label: "Calories",
          field: "calories",
          sortable: true,
        },
        { name: "fat", label: "Fat (g)", field: "fat", sortable: true },
        { name: "carbs", label: "Carbs (g)", field: "carbs" },
        { name: "protein", label: "Protein (g)", field: "protein" },
        { name: "sodium", label: "Sodium (mg)", field: "sodium" },
        {
          name: "calcium",
          label: "Calcium (%)",
          field: "calcium",
          sortable: true,
          sort: (a, b) => parseInt(a, 10) - parseInt(b, 10),
        },
        {
          name: "iron",
          label: "Iron (%)",
          field: "iron",
          sortable: true,
          sort: (a, b) => parseInt(a, 10) - parseInt(b, 10),
        },
      ],
      where: [],
      where_options: [
        { label: "Northern America", value: "na" },
        { label: "Latin America", value: "la" },
        { label: "Europe", value: "eu" },
        { label: "Africa", value: "af" },
        { label: "Middle East", value: "me" },
        { label: "Asia", value: "as" },
        { label: "Oceania", value: "oc" },
        { label: "Space", value: "sp" },
        { label: "United States", value: "usa" },
        { label: "European union", value: "euu" },
        { label: "Not in Schengen", value: "nis" },
      ],
      how: [],
      how_options: [
        { label: "Fun", value: "fun" },
        { label: "Near beach", value: "near_beach" },
        { label: "Friendly people", value: "friendly_people" },
        { label: "Safe for women", value: "safe_for_women" },
      ],
      health: [],
      health_options: [
        { label: "Clean air annually", value: "clean_air_annually" },
        { label: "Great hospitals", value: "great_hospitals" },
        { label: "No smoking", value: "no_smoking" },
        { label: "Not overweight", value: "not_overweight" },
        { label: "Not obese", value: "not_obese" },
      ],
      size: [],
      size_options: [
        { label: "Town", value: "town" },
        { label: "Large town", value: "large" },
        { label: "City", value: "city" },
        { label: "Metropolis", value: "metropolis" },
        { label: "Megacity", value: "megacity" },
      ],
      climate: [],
      climate_options: [
        { label: "Tropical climate", value: "tropical" },
        { label: "Desert climate", value: "desert" },
        { label: "Temperate climate", value: "temperate" },
        { label: "Continental climate", value: "continental" },
        { label: "Polar climate", value: "polar" },
        { label: "Good UV Index", value: "uv" },
        { label: "Good humidity", value: "humidity" },
      ],
      cost: [],
      cost_options: [{ label: "Safe", value: "safe" }],
      development: [],
      development_options: [
        { label: "Safe", value: "safe" },
        { label: "Fun", value: "fun" },
      ],
      quality: [],
      quality_options: [
        { label: "Safe", value: "safe" },
        { label: "Fun", value: "fun" },
      ],
      geo: [],
      geo_options: [
        { label: "Safe", value: "safe" },
        { label: "Fun", value: "fun" },
      ],
      culture: [],
      culture_options: [
        { label: "Safe", value: "safe" },
        { label: "Fun", value: "fun" },
      ],
      timezone: [],
      timezone_options: [
        { label: "Safe", value: "safe" },
        { label: "Fun", value: "fun" },
      ],
      language: [],
      language_options: [
        { label: "Safe", value: "safe" },
        { label: "Fun", value: "fun" },
      ],
    };
  },
  methods: {
    imageURL(image) {
      return require("assets/images/" +
        image.replace("/assets/img/places/", "", -1));
    },
    safetyColor(s) {
      if (s <= 2) {
        return "red";
      } else if (s >= 2 && s <= 4) {
        return "yellow";
      }

      return "green";
    },
    filterCountry(val, update) {
      if (val === "") {
        update(() => {
          this.countries = countries;

          // with Quasar v1.7.4+
          // here you have access to "ref" which
          // is the Vue reference of the QSelect
        });
        return;
      }

      update(() => {
        const needle = val.toLowerCase();
        this.countries = countries.filter(
          (v) => v.toLowerCase().indexOf(needle) > -1
        );
      });
    },
  },
};
</script>

<style lang="scss">
.cities {
  // max-width: 1280px;
  // margin: 0 auto;
  // max-height: 95vh;
  // height: 400px;
}

.city {
  $size: 350px;
  .q-img {
    min-width: $size;
    min-height: $size;
    max-height: $size;
    background-size: cover;

    .q-img__content > div {
      padding: 8px;
    }
  }
}

.section-ranking {
  // background: #999;
}

.section-guide {
  // background: #ddd;
}

.section-cost {
  // background: #333;
  // color: #fff;
}
.section-proscons {
  // background: red;
}
</style>
