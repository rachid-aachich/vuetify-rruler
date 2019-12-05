<template>
  <v-app>
    <v-card
      style="padding: 10px;">

      <v-row>
        <v-col class="d-flex" cols="10" sm="6">
          <v-select
            v-model="repeatSelect"
            :items="repeatItems"
            item-text="text"
            item-value="value"
            label="Repeat"
            :change="onChange()"
          ></v-select>
        </v-col>

        <v-col class="d-flex" cols="10" sm="6">
          <v-text-field
            v-model="interval"
            hide-details
            label="Interval"
            min="1"
            single-line
            :change="onChange()"
            type="number"
          />
        </v-col>
      </v-row>

      <v-row>
        <v-col
          v-if="repeatSelect == 1"
          class="d-flex"
          cols="10"
          sm="6">
          <v-row
            style="padding: 20px;">

            <span style="display: flex; align-items: center; justify-content: center; font-size: smaller;">Repeat on</span>

            <v-avatar
              v-for="(day, i) in dayChips"
              :key="i"
              left
              @click="toggleDay(day)"
              :model="day"
              :color="day.color"
              @mouseover="dayHoverIn(day)"
              @mouseleave="dayHoverOut(day)"
              style="cursor: pointer; margin: 5px;">
                {{ day.name }}
            </v-avatar>

          </v-row>
        </v-col>

        <v-col
          v-if="repeatSelect == 2"
          class="d-flex"
          cols="10"
          sm="6">

          <v-row>

            <v-col class="d-flex">

              <v-radio-group v-model="radioGroup">
                <v-radio
                  label="Day of month"
                  value="day_month"
                  >
                </v-radio>

                <v-radio
                  label="Day of week"
                  value="day_week"
                  >
                </v-radio>
              </v-radio-group>

            </v-col>

            <v-col class="d-flex">

              <v-text-field
                v-if="radioGroup == 'day_month'"
                v-model="day_month"
                hide-details
                :change="onChange()"
                label="Date"
                single-line
                min="1"
                max="31"
                type="number"/>

              <v-row
                v-if="radioGroup == 'day_week'">

                <v-col class="d-flex">
                  <v-select
                    v-model="occruencePosition"
                    :items="dayOccurenceList"
                    item-text="text"
                    item-value="value"
                    :change="onChange()"
                    label="Day position"
                  ></v-select>
                </v-col>

                <v-col class="d-flex">
                  <v-select
                    v-model="weekDayOccurence"
                    :items="dayList"
                    :change="onChange()"
                    label="Day"
                  ></v-select>
                </v-col>
              </v-row>

            </v-col>

          </v-row>

        </v-col>

        <v-col class="d-flex" cols="12" sm="6">
          <v-row>

            <v-col class="d-flex" cols="12" sm="6">
              <v-select
                v-model="endSelect"
                :items="end"
                item-text="text"
                item-value="value"
                :change="onChange()"
                label="End"
              ></v-select>
            </v-col>

            <v-col
              v-if="endSelect == 2"
              class="d-flex"
              cols="12"
              sm="6">
              <v-text-field
                v-model="count"
                hide-details
                :change="onChange()"
                label="Count"
                single-line
                min="1"
                type="number"/>
            </v-col>

            <v-col
              v-if="endSelect == 1"
              cols="11"
              sm="5">
              <v-dialog
                ref="dialog"
                v-model="endDateModal"
                :return-value.sync="endDate"
                persistent
                full-width
                :change="onChange()"
                width="290px">
                <template v-slot:activator="{ on }">
                  <v-text-field
                    v-model="endDate"
                    label="Select a date"
                    prepend-icon="event"
                    readonly
                    v-on="on"
                  ></v-text-field>
                </template>
                <v-date-picker
                  v-model="endDate"
                  locale="fr"
                  scrollable>
                  <div class="flex-grow-1"></div>
                  <v-btn text color="primary" @click="endDateModal = false">Cancel</v-btn>
                  <v-btn text color="primary" @click="$refs.dialog.save(endDate)">Ok</v-btn>
                </v-date-picker>
              </v-dialog>
            </v-col>

          </v-row>
        </v-col>

      </v-row>
    </v-card>
  </v-app>
</template>

<style>
  button {
    color: 'red';
  }

  button:hover {
    color: 'black';
  }
</style>

<script>

import { RRule } from 'rrule'

export default
{
  
  name: 'App',
  data()
  {
    return {
      repeatItems: [
        {text: "Daily", value: 0},
        {text: "Weekly", value: 1},
        {text: "Monthly", value: 2}
      ],
      repeatSelect: 0,
      interval: 1,
      maxRules: [
        (v)=> {
          if (v > 0) {
            return 'Error'
          }
        }
      ],
      dayChips: [
        {name: 'MO', value: RRule.MO, color: 'red', toggle: true},
        {name: 'TU', value: RRule.TU, color: 'grey', toggle: false},
        {name: 'WE', value: RRule.WE, color: 'grey', toggle: false},
        {name: 'TH', value: RRule.TH, color: 'grey', toggle: false},
        {name: 'FR', value: RRule.FR, color: 'grey', toggle: false},
        {name: 'SA', value: RRule.SA, color: 'grey', toggle: false},
        {name: 'SU', value: RRule.SU, color: 'grey', toggle: false}
      ],
      colors: {
        darkGrey: '#696767',
        lightGrey: '#bfbaba',
        darkRed: '#ad0505',
        lightRed: '#f94a4a'
      },
      end: [
        {text: "Never", value: 0},
        {text: "Until", value: 1},
        {text: "Count", value: 2}
      ],
      endSelect: 0,
      endDate: null,
      endDateModal: false,
      count: 1,
      radioGroup: null,
      day_month: null,
      day_week: null,
      dayOccurenceList: [
        {text: "First", value: 1},
        {text: "Second", value: 2},
        {text: "Third", value: 3},
        {text: "Fourth", value: 4},
        {text: "Last", value: -1}
      ],
      dayList: [
        {text: "Monday", value: RRule.MO},
        {text: "Tuesday", value: RRule.TU},
        {text: "Wednesday", value: RRule.WE},
        {text: "Thursday", value: RRule.TH},
        {text: "Friday", value: RRule.FR},
        {text: "Saturday", value: RRule.SA},
        {text: "Sunday", value: RRule.SU}
      ],
      rruleString: 'RRULE:FREQ=DAILY;INTERVAL=1',
      rruleObject: null,
      weekDays: [RRule.MO],
      weekDayOccurence: RRule.MO,
      occruencePosition: 1,
      requiredRule: [
        v => !!v || "Field is required"
      ]
    }
  },

  methods: {

    toggleDay(day) {
      let scope = this;

      if(scope.weekDays.length == 1 && day.toggle)
        return false;

      day.color = day.toggle ? 'grey' : 'red'
      day.toggle = !day.toggle;

      let index = scope.findElementInArray(scope.weekDays, day.value);
      if(index >= 0)
        scope.weekDays.splice(index, 1);
      else
        scope.weekDays.push(day.value);

      scope.onChange();
    },

    findElementInArray(array, element) {
      let ret = -1;

      for(var i = 0; i < array.length; i++) {
        if (array[i] == element) {
          ret = i;
          break;
        }
      }

      return ret;
    },

    dayHoverIn(day) {
      day.color = day.toggle ? this.colors.darkRed : this.colors.darkGrey;
    },

    dayHoverOut(day) {
      day.color = day.toggle ? this.colors.lightRed : this.colors.lightGrey;
    },

    onChange() {
      let scope = this;

      let freq, byweekday, bymonthday, bysetpos, until, count;

      if(scope.interval === '')
        scope.interval = 1;
      
      switch(scope.repeatSelect) {
        default:
        case 0:
          freq = RRule.DAILY;
          break;
        case 1:
          freq = RRule.WEEKLY;
          byweekday = scope.weekDays;
          break;
        case 2:
          freq = RRule.MONTHLY;
          if(scope.radioGroup == 'day_month') {
            bymonthday = scope.day_month;
          } else {
            byweekday = scope.weekDayOccurence;
            bysetpos = scope.occruencePosition;
          }
          break;
      }

      switch(scope.endSelect) {
        default:
        case 0:
          break;
        case 1:
          until = scope.endDate;
          break;
        case 2:
          count = scope.count;
          break;
      }

      let rruleObject = new RRule({
        freq: freq,
        byweekday: byweekday,
        bymonthday: bymonthday,
        bysetpos: bysetpos,
        until: until,
        count: count,
        interval: scope.interval
      })

      
      scope.rruleString = rruleObject.toString();
      scope.$emit('onChange', rruleObject);
    }

  }

}

</script>