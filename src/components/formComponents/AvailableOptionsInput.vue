<template>
  <div class="wrapper">
    <label :for="inputName" class="genericLabel">{{ labelName }}</label>
    <input
      type="texte"
      :id="inputName"
      class="genericInput"
      autocomplete="false"
      :name="inputName"
      :value="value"
      @input="$emit('input', $event.target.value)"
      @focus="displayDataList"
      @blur="hideDataList"
    />
    <datalist id="availableTechnologies">      
      <option v-for="(opt, index) in dataList" :key="index" @click="addTechno" tabindex="-1">{{ opt }}</option>
    </datalist>
  </div>
</template>

<script>
export default {
  name: 'AvailableOptionsInput',
  props: {
    labelName: String,
    value: {
      type: String,
      default: ''
    },
    dataList: {
      type: Set
    }
  },
  data() {
    return {
      inputName: this.labelName.normalize('NFD').replace(/[\u0300-\u036f]/g, "")
    }
  },
  methods: {
    displayDataList(event) {
      // Affichage des sugestions de technologies 
      const datalist = document.getElementById('availableTechnologies')
      datalist.style.display = 'block'
      datalist.style.width = event.target.offsetWidth + 'px'
      datalist.style.left = event.target.offsetLeft + 'px'
      datalist.style.top = event.target.offsetTop + event.target.offsetHeight + 'px'
    },
    hideDataList(event) {
      // Masquage des sugestions de technologies 
      if (event.relatedTarget === null || event.relatedTarget.tagName !== 'OPTION')
        document.getElementById('availableTechnologies').style.display = 'none'
    },
    addTechno(event) {
      // Ajout de la technologie sélectionnée
      const input = document.getElementById(this.inputName)
      if (input.value === '') {
        input.value = event.target.value
      } else {
        input.value += ', ' + event.target.value
      }
      document.getElementById('availableTechnologies').style.display = 'none'
      input.dispatchEvent(new InputEvent('input'))
    }
  }
}
</script>

<style scoped>
datalist {
  position: absolute;
  background-color: var(--main-text-color);
  font-family: sans-serif;
  font-size: 0.8rem;
  box-shadow: 3px 7px 9px rgb(21 21 21 / 78%);
  display:none;
}

option {
  color: var(--main-bg-color);
  padding: 4px;
  margin-bottom: 1px;
  cursor: pointer;
}

option:hover {
  background-color: var(--main-lighter-violet);
}
</style>