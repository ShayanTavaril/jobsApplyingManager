<template>
  <div class="shadowBox" name="shadowBox" @click="(e) => { if(e.target.classList[0] === 'shadowBox') $emit('close') }">
    <div class="popin shadow" :id="piste.id">
      <h1 v-if="isNewItem">Création de la piste</h1>
      <h1 v-else>Détails de la piste</h1>
      <form @submit="checkForm">
        <div class="wholePiste">
          <div class="left">
            <TextInput inputType="text" labelName="Société" v-model="piste.societe" isRequired></TextInput>
            <AvailableOptionsInput labelName="Technologies" v-model="piste.technos" :dataList="listTechno"></AvailableOptionsInput>
            <TextInput inputType="url" labelName="Liens" v-model="piste.liens"></TextInput>
            <DatesManager v-model="piste.dates" @change="updateDatesList"></DatesManager>
          </div>
          <div class="right">
            <TextInput inputType="text" labelName="Etat" isDisabled v-model="piste.etat"></TextInput>
            <TextInput inputType="text" labelName="Interlocuteur" v-model="piste.interlocuteur"></TextInput>
            <TextInput inputType="text" labelName="Localisation" v-model="piste.localisation"></TextInput>
            <CheckboxInput v-if="!isNewItem" labelName="Fermée" v-model="piste.closed"></CheckboxInput>
          </div>
        </div>
        <div class="middle">
          <TextInput inputType="textarea" labelName="Notes" v-model="piste.notes"></TextInput>
        </div>
        <div class="histContainer" v-if="piste.historique">
          <h2>Historique</h2>
          <div class="histSubContainer">
            <HistoriqueLine v-for="histo in piste.historique" :key="histo.date.toString()" :histo="histo"></HistoriqueLine>
          </div>
        </div>
        <div class="buttonContainer">
          <button v-if="!isNewItem" @click="$emit('delete', piste)" class="deleteButton">Supprimer</button>
          <button v-if="isNewItem" type="submit">Enregistrer</button>
          <button v-else type="submit">Mettre à jour</button>
          <button @click="$emit('close')">Annuler</button>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
import DatesManager from './DatesManager.vue'
import TextInput from './formComponents/TextInput.vue'
import AvailableOptionsInput from './formComponents/AvailableOptionsInput.vue'
import CheckboxInput from './formComponents/CheckboxInput.vue'
import HistoriqueLine from './tableLine/HistoriqueLine.vue'
import { createUUID, deepCopy, deepComparison } from "../common.js"

export default {
  name: 'PopIn',
  components: {
    DatesManager, TextInput, AvailableOptionsInput, CheckboxInput, HistoriqueLine
  },
  props: {
    isNewItem: {
      type: Boolean
    },
    listTechno: {
      type: Set
    },
    pisteToModify: {
      type: Object,
      default: () => {
        return {
          id: createUUID(),
          etat: "Nouvelle",
          societe: "",
          interlocuteur: "",
          localisation: "",
          technos: "",
          liens: "",
          notes: "",
          historique: null,
          closed: false,
          dates: []
        }
      }
    }
  },
  data() {
    return {
      piste: deepCopy(this.pisteToModify)
    }
  },
  mounted() {
    // Stop le fire d'un des boutons du formulaire lors de l'appui sur la touche ENTREE dans un input
    document.querySelectorAll('.popin input').forEach(input => {
      input.addEventListener('keypress', e => {
        if (e.keyCode === 13) e.preventDefault() 
        })
    });
  },
  methods: {
    updateDatesList(newDates) {
      this.piste.dates = newDates
    },
    checkForm(e) {
      // Vérification du formulaire et mise à jour de l'historique avant sauvegarde
      e.preventDefault();

      // Gestion de l'historique 
      if (this.isNewItem) {
        this.piste.historique = [{
          date: new Date().toISOString().split('T')[0] + 'T' + new Date().toLocaleTimeString(),
          modif: "Création de la piste."
        }]
      }
      else {
        const modifs = deepComparison(this.piste, this.pisteToModify)
        let singPluriel = "du champ"
        
        if (modifs.length > 0) {
          if (modifs.length > 1) singPluriel = "des champs"
          this.piste.historique.push({
            date: new Date().toISOString().split('T')[0] + 'T' + new Date().toLocaleTimeString(),
            modif: `Modification ${singPluriel} : ${modifs}`
          })
        }     
      }

      this.$emit('save', this.piste)
    }
  }
}
</script>

<style scoped>
.shadowBox {
  background: rgb(121 121 121 / 52%);
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1999;
  overflow: auto;
}
.popin {
  margin: 40px;
  background: var(--main-text-color);
  border: none;
  border-radius: 7px;
}
h1 {
  margin: 0;
  padding: 0.3em 0;
  font-size: 1.6em;
  border-top-left-radius: 7px;
  border-top-right-radius: 7px;
  background-color: var(--main-violet);
}
.wholePiste {
  display: flex;
}
.wholePiste > div {
  display: flex;
  flex-flow: column wrap;
  padding: 10px;
}
.left {
  flex: 3 1 auto;
}
.right {
  flex: 1 1 auto;
}
.left > div,
.right > div {
  margin: 5px 0;
}
h1 {
  text-align: center;
}
h2 {
  margin: 0 0 10px;
  color: var(--main-bg-color);
}
.histContainer,
.middle {
  padding: 10px;
}
.histSubContainer {
  padding: 5px;
  background-color: rgba(239, 239, 239, 0.3);
  border: 1px solid rgba(118, 118, 118, 0.3);
}
.buttonContainer {
  display: flex;
  justify-content: flex-end;
  padding: 10px;
}
button {
  margin-left: 10px;
}

@media screen and (max-width: 900px) {
  .popin {
    margin: 25px;
  }
  .wholePiste {
    display: flex;
    flex-flow: column wrap;
  }
}
</style>