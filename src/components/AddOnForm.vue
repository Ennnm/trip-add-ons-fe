<template>
  <div class="container">
    <div class="addon-card card">
      <h2>Add-ons</h2>
      <div v-for="(addon, index) in addons" :key="index" :class="{ 'muted': isConflictingToSelection(addon) }"
        class="item">
        <div class="addon-info">
          <p style="font-weight: bold;">{{ addon.name }}</p>
          <p v-if="addon.start_time && addon.end_time">
            Time: {{ formatTime(addon.start_time) }} - {{ formatTime(addon.end_time) }}
          </p>
          <p>Price: USD {{ addon.cost }}</p>
        </div>
        <input type="checkbox" v-model="selectedAddons" :value="addon" @change="onSelect"
          :disabled="isConflictingToSelection(addon)">
      </div>
    </div>
    <div class="cost-card card">
      <h2>Total Cost</h2>
      <div class="item">Total Cost: ${{ totalCost }}</div>
      <button @click="submitSelectedAddons">Submit Selected Add-ons</button>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue'
// TODO: replace with deployed url
const baseUrl = 'http://localhost:3000'

export default {
  name: 'AddOnForm',
  props: {
    msg: String
  },
  data() {
    return {
      addons: [],
      selectedAddons: ref([]),
      totalCost: 0,
      conflictSet: new Set()
    };
  },
  mounted() {
    this.fetchAddons();
  },
  methods: {
    isTimeInRange(time, startTime, endtime) {
      return time > startTime && time < endtime
    }, addConflictSetToAddOns(addOns) {
      addOns = addOns.map((v) => { return { ...v, conflictSet: new Set() } })
      for (let i = 0; i < addOns.length; i++) {
        let addOn = addOns[i]
        if (!addOn.start_time && !addOn.end_time) {
          continue
        }
        for (let j = i + 1; j < addOns.length; j++) {
          let otherAddOn = addOns[j]
          if (!otherAddOn.start_time && !otherAddOn.end_time) {
            continue
          }
          if (addOn.start_time == otherAddOn.start_time && addOn.end_time == otherAddOn.end_time) {
            addOn.conflictSet.add(otherAddOn.id)
            otherAddOn.conflictSet.add(addOn.id)
          } else {
            if (this.isTimeInRange(addOn.start_time, otherAddOn.start_time, otherAddOn.end_time) ||
              this.isTimeInRange(addOn.end_time, otherAddOn.start_time, otherAddOn.end_time)) {
              addOn.conflictSet.add(otherAddOn.id)
              otherAddOn.conflictSet.add(addOn.id)
            }
          }
        }
      }
      return addOns
    },
    async fetchAddons() {
      try {
        const response = await fetch(`${baseUrl}/add_ons`);
        const data = await response.json();
        const formatted = data.map((v) => {
          if (v.start_time) {
            v = { ...v, start_time: new Date(v.start_time) }
          }
          if (v.end_time) {
            v = { ...v, end_time: new Date(v.end_time) }
          }
          return v
        })
        this.addons = this.addConflictSetToAddOns(formatted);
        console.log(this.addons)
      } catch (error) {
        console.error('Error fetching add-ons:', error);
      }
    },
    calculateTotalCost() {
      this.totalCost = this.selectedAddons.reduce((total, addOn) => {
        return total + addOn.cost
      }, 0);
    },
    calculateConflicted() {
      this.conflictSet = new Set()
      this.selectedAddons.forEach((a) => {
        if (a.conflictSet.size > 0) {
          this.conflictSet.add(...a.conflictSet)
        }
      })
      console.log(this.conflictSet)
    },
    isConflictingToSelection(addOn) {
      return this.conflictSet.has(addOn.id)
    },
    onSelect() {
      this.calculateTotalCost()
      this.calculateConflicted()
    },
    formatTime(date) {
      const hours = date.getHours().toString().padStart(2, '0');
      const minutes = date.getMinutes().toString().padStart(2, '0');
      return `${hours}.${minutes}`;
    },
    async submitSelectedAddons() {
      try {
        const response = await fetch(`${baseUrl}/trip_add_ons`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },

          body: JSON.stringify({
            trip_id: this.tripId,
            add_on_ids: this.selectedAddons.map((v) => v.id),
          }),
        });
        if (!response.ok) {
          throw new Error('Network response was not ok ', response.status);
        }

        const data = await response.json();
        // refresh the form 
        console.log(data); // Handle the response as needed
      } catch (error) {
        console.error('Error submitting selected addons:', error);
      }
    },
  },
};
</script>

<style scoped>
.container {
  display: flex;
  margin: 20px;
  /* Add margin between the cards */
}

.card {
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 16px;
  margin: 20px;
  text-align: left;
}

.card h2 {
  margin: 0;
  margin-bottom: 6px;
}

.card p {
  margin: 0;
  margin-bottom: 6px;
}

.card input {
  margin-left: 0px;
  margin-right: 16px;
}

.addon-card {
  flex: 2;
  /* Take up 2/3 of the available space */
}

.cost-card {
  flex: 1;
  /* Take up 1/3 of the available space */
}

.item {
  flex-direction: row;
  margin-bottom: 8px;
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
}

.muted {
  color: lightgray
}

.addon-info {
  display: flex;
  flex-direction: column;
}
</style>