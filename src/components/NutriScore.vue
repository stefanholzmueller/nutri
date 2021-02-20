<template>
  <v-container>
    <h1 align="center">NutriScore Rechner</h1>
    <v-row justify="center">
      <v-col cols="12" sm="9" md="6" lg="4">
        <v-form v-model="isValid">
          <v-row align="center">
            <v-radio-group v-model="solidFoodOrBeverage" row>
              <v-radio label="Lebensmittel" value="solidFood"></v-radio>
              <v-radio label="Getränke" value="beverage" disabled></v-radio>
            </v-radio-group>
          </v-row>
          <p>Nährwerte pro 100 Gramm</p>
          <v-row align="center">
            <v-col cols="5">
              <v-text-field v-model="kcal" label="Energie (kcal)" :rules="numberRules"/>
            </v-col>
            <v-col cols="5">
              <v-text-field v-model="kj" label="Energie (kJ)" type="number" min="0" disabled/>
            </v-col>
            <v-col cols="1" offset="1" class="red--text" v-if="energyPoints">
              {{ energyPoints }}
            </v-col>
          </v-row>
          <v-row align="center">
            <v-col>
              <v-text-field v-model="sugars" label="Zucker (g)" type="number" min="0" max="100"/>
            </v-col>
            <v-col cols="1" offset="1" class="red--text">
              {{ sugarsPoints }}
            </v-col>
          </v-row>
          <v-row align="center">
            <v-col>
              <v-text-field v-model="fats" label="Gesättigte Fettsäuren (g)" type="number" min="0" max="100"/>
            </v-col>
            <v-col cols="1" offset="1" class="red--text">
              {{ fatsPoints }}
            </v-col>
          </v-row>
          <v-row align="center">
            <v-col cols="5">
              <v-text-field v-model="salt" label="Salz (g)" type="number" min="0"/>
            </v-col>
            <v-col cols="5">
              <v-text-field v-model="sodium" label="Natrium (mg)" type="number" min="0" disabled/>
            </v-col>
            <v-col cols="1" offset="1" class="red--text">
              {{ sodiumPoints }}
            </v-col>
          </v-row>
          <v-row align="center">
            <v-col>
              <v-text-field v-model="protein" label="Protein (g)" type="number" min="0" max="100"/>
            </v-col>
            <v-col cols="1" offset="1" class="green--text">
              {{ proteinPoints }}
            </v-col>
          </v-row>
          <v-row align="center">
            <v-col>
              <v-text-field v-model="fibers" label="Ballaststoffe (g)" type="number" min="0" max="100"/>
            </v-col>
            <v-col cols="1" offset="1" class="green--text">
              {{ fibersPoints }}
            </v-col>
          </v-row>
          <v-row align="center">
            <v-col>
              <v-select v-model="fruits" :items="fruitsOptions" label="Obst, Gemüse, Nüsse (%)"/>
            </v-col>
            <v-col cols="1" offset="1" class="green--text">
              {{ fruitsPoints }}
            </v-col>
          </v-row>
          <v-row v-if="isValid">
            <v-col>
              Gesamtpunkte
            </v-col>
            <v-col cols="1" offset="1" v-if="points">
              {{ points }}
            </v-col>
          </v-row>
          <v-row v-if="isValid">
            <v-col>
              NutriScore
            </v-col>
            <v-col v-if="points">
              {{ score }}
            </v-col>
          </v-row>
        </v-form>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
function findPoints(array, n) {
  const index = array.findIndex(threshold => n < threshold);
  return index < 0 ? NaN : index;
}

export default {
  data: function () {
    return {
      isValid: false,
      solidFoodOrBeverage: "solidFood",
      kcal: "",
      kj: null,
      sugars: null,
      fats: null,
      salt: null,
      sodium: null,
      protein: null,
      fibers: null,
      fruits: "bis 40%",
      fruitsOptions: [ "bis 40%", "mehr als 40%", "mehr als 60%", "mehr als 80%" ],
      numberRules: [
        input => !Number.isNaN(Number.parseFloat(input.replace(",", ".")))
      ]
    };
  },
  computed: {
    energyPoints: function() {
      const kcal = Number.parseFloat(this.kcal.replace(",", "."));
      return findPoints([ 80.05, 160.05, 240.05, 320.05, 400.05, 480.05, 560.05, 640.05, 720.05, 800.05, Infinity ], this.kcal);
    },
    sugarsPoints: function() {
      return [ 4.505, 9.05, 13.505, 18.05, 22.505, 27.05, 31.05, 36.05, 40.05, 45.05, Infinity ].findIndex(threshold => this.sugars < threshold);
    },
    fatsPoints: function() {
      return [ 1.05, 2.05, 3.05, 4.05, 5.05, 6.05, 7.05, 8.05, 9.05, 10.05, Infinity ].findIndex(threshold => this.fats < threshold);
    },
    sodiumPoints: function() {
      const sodium = this.salt * 400;
      return [ 90.05, 180.05, 270.05, 360.05, 450.05, 540.05, 630.05, 720.05, 910.05, 900.05, Infinity ].findIndex(threshold => sodium < threshold);
    },
    proteinPoints: function() {
      return [ 1.605, 3.205, 4.805, 6.405, 8.005, Infinity ].findIndex(threshold => this.protein < threshold);
    },
    fibersPoints: function() {
      return [ 0.905, 1.905, 2.805, 3.705, 4.705, Infinity ].findIndex(threshold => this.fibers < threshold);
    },
    fruitsPoints: function() {
      switch (this.fruits) {
        case "bis 40%": return 0;
        case "mehr als 40%": return 1;
        case "mehr als 60%": return 2;
        case "mehr als 80%": return 5;
      }
    },
    points: function() {
      return this.energyPoints + this.sugarsPoints + this.fatsPoints + this.sodiumPoints - this.proteinPoints - this.fibersPoints - this.fruitsPoints;
    },
    score: function() {
      if (this.points < 0) return "A (-15 bis -1 Punkte)";
      else if (this.points < 3) return "B (0 bis 2 Punkte)";
      else if (this.points < 11) return "C (3 bis 10 Punkte)";
      else if (this.points < 19) return "D (11 bis 18 Punkte";
      else return "E (19 bis 40 Punkte)";
    }
  }
}
</script>
