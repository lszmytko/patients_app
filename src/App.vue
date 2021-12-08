<template>
  <div>
    <h1>Patients</h1>
    <div class="buttons_container">
      <button @click="handleFilterAgeActive">
        {{ filterAgeActive ? "Unfilter" : "Filter" }} age >30
      </button>
      <button @click="handleFilterAgeAndMedicineActive">
        {{ filterAgeAndMedicineActive ? "Unfilter" : "Filter" }} age &lt; 63
        and medicines strength > 8
      </button>
    </div>
    <div class="card-container">   
        <Card
          v-for="patient in patientsToShow"
          :key="patient.id"
          :name="patient.name"
          :lastName="patient.lastName"
          :medicines="patient.medicines"
          :age="patient.age"
            />         
    </div>
  </div>
</template>

<script>
import { watch, ref, computed } from "vue";
import axios from "axios";
import Card from "./components/Card.vue";

export default {
  name: "App",
  components: { Card },
  setup() {

    // Refs
    let patients = ref(null);
    let patientsAbove30 = ref(null);
    let patientsBelow63withMedicineStrength = ref(null);
    let filterAgeActive = ref(false);
    let filterAgeAndMedicineActive = ref(false);

    // Constants

    const ageCriteria = 30;
    const ageAndMedicineCriteria = {
      age: 63,
      medicineStrength: 8
    }

    // Handling filter criteria

    function handleFilterAgeActive() {
      this.filterAgeActive = !this.filterAgeActive;
      if (this.filterAgeAndMedicineActive) {
        this.filterAgeAndMedicineActive = false;
      }
    }

    function handleFilterAgeAndMedicineActive() {
      this.filterAgeAndMedicineActive = !this.filterAgeAndMedicineActive;
      if (this.filterAgeActive) {
        this.filterAgeActive = false;
      }
    }

    // Initial fetch

    axios
      .all([
        axios.get("https://cerber.pixel.com.pl/api/patients"),
        axios.get("https://cerber.pixel.com.pl/api/medicine"),
      ])
      .then(
        axios.spread(function (response1, response2) {
          const patientsFetched = response1.data.map((patient) => {
            patient.medicines = [];
            for (const medicine of response2.data) {
              if (medicine.patientIds.includes(patient.id)) {
                patient["medicines"].push(medicine);
              }
            }
            return patient;
          });
          patients.value = patientsFetched;
        })
      )
      .catch((e) => console.log(e));

    // Patients watcher

    watch(patients, () => {

      // Generate above30 array
      patientsAbove30.value = patients.value.filter((patient) => {
        return patient.age > ageCriteria;
      });

      // turning array of proxies into normal array
      const tempArray = [...patients.value].map((item) => {
        return JSON.parse(JSON.stringify(item));
      });

      // Generate array of below 63 and 

      patientsBelow63withMedicineStrength.value = tempArray
        .map((patientTemp) => {
          patientTemp.medicines = patientTemp.medicines.filter((medicine) => {
            return medicine.strength > ageAndMedicineCriteria.medicineStrength;
          });
          return patientTemp;
        })
        .filter((patientTemp) => {
          return patientTemp.age < ageAndMedicineCriteria.age;
        });
    });

  // Generate final array to show
   const patientsToShow = computed(function(){
     if(filterAgeActive.value === true){
       return patientsAbove30.value;
     } else if(filterAgeAndMedicineActive.value === true){
       return patientsBelow63withMedicineStrength.value;
     }
     return patients.value;
   })

    return {
      patients,
      patientsToShow,
      filterAgeActive,
      filterAgeAndMedicineActive,
      handleFilterAgeActive,
      handleFilterAgeAndMedicineActive,
      patientsAbove30,
      patientsBelow63withMedicineStrength,
    };
  },
};
</script>

<style>
#app {
  font-family: "Libre Baskerville", Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  height: 100%;
  width: 100%;
}

h1 {
  color: #2b2d42;
  padding: 1.5rem;
}

.card-container {
  display: grid;
  grid-template-columns: repeat(auto-fill, 300px);
  justify-content: center;
  width: 100%;
  gap: 2rem;
}

.buttons_container {
  display: flex;
  justify-content: center;
  gap: 1rem;
  margin-bottom: 2rem;
  padding: 0 2rem;
}

ul {
  padding: 0;
}

button {
  background-color: rgb(142, 202, 230);
  border: 1px solid transparent;
  border-radius: .5rem;
  box-sizing: border-box;
  cursor: pointer;
  display: inline-block;
  font-size: .75rem;
  font-weight: 400;
  line-height: 1.15385;
  margin: 0;
  outline: none;
  padding: 8px 0.8em;
  text-align: center;
  text-decoration: none;
  letter-spacing: 1.5px;
}

button:hover,
button:focus {
  background-color: #219ebc;
}

button:focus {
  box-shadow: 0 0 0 4px rgba(0, 149, 255, 0.15);
}

@media screen and (min-width: 768px){
  button{
    font-size: 1rem;
  }
}

</style>
