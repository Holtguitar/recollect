<template>
  <!-- Normal View -->
  <div v-if="!editMode" class="scene scene--card" :style="this.displayStyle">
    <div
      class="card"
      v-bind:class="{ flipme: cardOne == 'flipped' }"
      :style="this.cardStyle"
      @click="getInfo($event)"
    >
      <div class="card__face card__face--front">
        <div v-if="(this.front.length < 100)" class="front-details">
          {{ front }}
        </div>
        <div v-else class="front-details-long">{{ front }}</div>
      </div>
      <div class="card__face card__face--back">
        <div v-if="(this.back.length < 100)" class="back-details">
          {{ back }}
        </div>
        <div v-else class="back-details-long">{{ back }}</div>
      </div>
    </div>
    <div class="card-controller" id={{id}}>
      <img
        src="\rotate-icon.png"
        class="rotate-image-icon"
        @click="
          cardOne == 'start' ? (cardOne = 'flipped') : (cardOne = 'start')
        "
        id="rotate-card"
      />
      <div id="{{id}}">
        <img
          src="\edit-icon.png"
          class="edit-icon"
          @click.prevent="this.editCard($event)"
        />
      </div>
      <div id="{{id}}">
        <img
          src="\delete-icon.png"
          class="delete-icon"
          @click.prevent="this.deleteCard($event)"
        />
      </div>
    </div>
  </div>
  <!-- Edit View -->
  <div v-else class="scene scene--card">
    <div
      class="card"
      :style="this.colorsEdit"
      v-bind:class="{ flipme: cardOne == 'flipped' }"
    >
      <div class="card__face card__face--front">
        <input
          class="front-details"
          type="text"
          placeholder="{{frontEdit}}"
          v-model="this.frontEdit"
          required
        />
      </div>
      <div class="card__face card__face--back">
        <input
          class="back-details"
          type="text"
          placeholder="{{backEdit}}"
          v-model="this.backEdit"
          required
        />
      </div>
    </div>
    <div class="card-controller">
      <img
        src="\rotate-icon.png"
        class="rotate-image-icon-edit"
        @click="
          cardOne == 'start' ? (cardOne = 'flipped') : (cardOne = 'start')
        "
      />
      <div class="text-color-edit">
        <img src="\letter-t.png" class="text-image" />
        <input
          type="color"
          class="text-color-picker"
          id="text-color-picker"
          @change="this.changeTextColor()"
        />
      </div>
      <div class="card-color-edit">
        <img src="\cards.png" class="cards-image" />
        <input
          type="color"
          class="card-color-picker"
          id="card-color-picker"
          @change="this.changeCardColor()"
        />
      </div>
      <div>
        <img
          src="\cancel-icon.png"
          class="cancel-icon"
          @click.prevent="this.cancelEdit()"
        />
      </div>
      <div>
        <img
          src="\save-icon.png"
          class="save-icon"
          @click.prevent="this.saveCard()"
        />
      </div>
    </div>
  </div>
</template>


<script>
import { getAuth } from '@firebase/auth'
import { getDatabase, ref, child, push, update, set } from '@firebase/database'
import { useStore } from 'vuex'

export default {
  props: [
    'subject',
    'email',
    'front',
    'back',
    'id',
    'colors',
    'textColor',
    'cardColor'
  ],

  data() {
    return {
      colorClass: `card color-${Math.floor(Math.random() * (6 - 1) + 1)}`,
      cardOne: 'start',
      user: getAuth(),
      store: useStore(),
      editMode: false,
      idEdit: null,
      subjectEdit: null,
      emailEdit: null,
      frontEdit: null,
      backEdit: null,
      textColorEdit: null,
      cardColorEdit: null,
      colorsEdit: null,
      fullsubject: null,
      fullemail: null,
      fullfront: null,
      fullback: null,
      fullid: null,
      fullcolors: null,
      fulldisplay: null,
      fulltextColor: null,
      fullcardColor: null,
     
    }
  },
  computed: {
    cardStyle() {
      return this.colors
    },
    displayStyle() {
      if (this.front === '' && this.back === '') {
        return 'display: none'
      }
    },
  },
  methods: {
    getInfo(e){
      console.log(e.target.__vueParentComponent.props)
    },
    changeCardColor() {
      let pick = document.getElementById("card-color-picker")
      this.cardColorEdit = pick.value
    },
    changeTextColor(){
      let pick = document.getElementById("text-color-picker")
      this.textColorEdit = pick.value
    },
    deleteCard(e) {
      let deleteConfirm = confirm('Are you sure you want to delete this card?')
      if (deleteConfirm) {
        const db = getDatabase()
        const id = this.id
        const subject = this.subject
        const userID = this.store.state.user.uid
        const path = `cards/${userID}/${subject}/${id}`
        const colors = this.colors
        const textColor = this.textColor
        const cardColor = this.cardColor
        const details = {
          db,
          path,
          userID,
          subject,
          id,
          colors,
          textColor,
          cardColor,
        }
        this.store.dispatch('deleteCard', details)
      }
    },
    editCard(e) {
      this.idEdit = this.id
      this.subjectEdit = this.subject
      this.emailEdit = this.email
      this.frontEdit = this.front
      this.backEdit = this.back
      this.cardColorEdit = this.cardColor
      this.textColorEdit = this.textColor
      this.colorsEdit = this.colors
      this.editMode = true
    },
    saveCard() {
      this.editMode = false
      const id = this.idEdit
      const subject = this.subjectEdit
      const email = this.store.state.user.email
      const front = this.frontEdit
      const back = this.backEdit
      const textColor = this.textColorEdit
      const cardColor = this.cardColorEdit
      const colors = `background-color: ${cardColor}; color: ${textColor}`
      const userID = this.store.state.user.uid
      const db = getDatabase()
      const path = `cards/${userID}/${subject}/${id}`
      const details = {
        path,
        userID,
        subject,
        id,
        email,
        front,
        back,
        db,
        colors,
        textColor,
        cardColor,
      }
      this.store.dispatch('editCard', details)
    },
    cancelEdit() {
      this.editMode = false
      this.idEdit = null
      this.subjectEdit = null
      this.emailEdit = null
      this.frontEdit = null
      this.backEdit = null
      this.colorsEdit = null
      this.cardColorEdit = null
      this.textColorEdit = null
    }
  },
}
</script>

<style scoped>
/* ------------------CARD STYLE--------------------- */
.scene {
  margin: 40px 0;
  perspective: 600px;
}

.card {
  width: 400px;
  height: 260px;
  transition: transform 1s;
  transform-style: preserve-3d;
  box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2);
}

.card:hover {
  cursor: pointer;
}

.card__face {
  position: fixed;
  width: 100%;
  height: 100%;
  text-align: center;
  font-weight: bold;
  font-size: 20px;
  backface-visibility: hidden;
  padding: 15px;
  overflow-y: scroll;
}

.card__face--back {
  pointer-events: none;
}

.flipme .card__face--back {
  pointer-events: auto;
}

.flipme .card__face--front {
  pointer-events: none;
}

.card-face::-webkit-scrollbar {
  display: none;
}

.flipme {
  transform: rotateY(180deg);
}

/* Front */

.front-details {
  margin-top: 25%;
}

.front-details::-webkit-scrollbar {
  display: none;
}

.front-details-long {
  margin-top: 0%;
}

.front-details-long::-webkit-scrollbar {
  display: none;
}

/* Back */

.card__face--back {
  position: fixed;
  top: 0%;
  transform: translateZ(-1px) rotateY(180deg);
}

.back-details {
  margin-top: 25%;
}

.back-details::-webkit-scrollbar {
  display: none;
}

.back-details-long {
  margin-top: 0%;
}

.back-details-long::-webkit-scrollbar {
  display: none;
}

/* --------------CARD CONTROLLER STYLE----------------------- */
/* Base Mode */
.edit-card {
  float: right;
  border: none;
  background-color: transparent;
  font-size: 20px;
  backface-visibility: hidden;
}

.rotate-image-icon-edit {
  width: 45px;
  height: 20px;
}

.edit-card__options {
  display: none;
}

.rotate-image-icon {
  width: 45px;
  height: 20px;
  margin-top: 5px;
}


#rotate-card:hover {
  cursor: pointer;
}

#rotate-card:active{
  transform: scale(.97);
}

.edit-icon {
  width: 25px;
  margin-top: 5px;
}

.delete-icon {
  width: 25px;
}

.card-controller {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  width: 400px;
  height: 50px;
  margin-top: 25px;
}

.card-controller button {
  border-radius: 10px;
  box-shadow: 0 1px 1px rgba(0, 0, 0, 0.25), 0 1px 1px rgba(0, 0, 0, 0.22);
  height: 25px;
  border-color: transparent;
  background-color: whitesmoke;
}

/* Edit Mode */

.edit-icon:hover,
.cancel-icon:hover,
.save-icon:hover,
.delete-icon:hover,
.text-image:hover,
.cards-image:hover,
.rotate-image-icon-edit:hover,
.text-color-picker:hover,
.card-color-picker:hover {
  cursor: pointer;
}

.cancel-icon {
  width: 25px;
}

.save-icon {
  width: 30px;
}

.text-color-picker,
.card-color-picker,
.cards-image,
.text-image {
  height: 25px;
}


.card-color-edit, .text-color-edit {
  width: 40px;
  display: grid;
}

.text-color-picker, .text-image {
  grid-column: 1;
  grid-row: 1;
}

.card-color-picker, .cards-image {
  grid-column: 1;
  grid-row: 1;
}
.text-color-picker{
  opacity: 0;
  width: 40px;
}

.card-color-picker {
  opacity: 0;
  width: 40px
}

.card-edit-menu {
  position: fixed;
  display: flex;
  flex-direction: column;
  height: 250px;
  width: 200px;
  border-radius: 15px;
  background-color: #0bd692;
  box-shadow: 0 14px 28px rgba(0, 0, 0, 0.25), 0 10px 10px rgba(0, 0, 0, 0.22);
  right: 0vw;
  top: 10%;
}

.card-controller button:hover {
  cursor: pointer;
}

.card-controller button:active {
  box-shadow: 0 4px 4px rgba(0, 0, 0, 0.25), 0 4px 4px rgba(0, 0, 0, 0.22);
}

@media (max-width: 900px) and (min-width: 350px) {
  /* --------CARD STYLE------------ */
  .card {
    width: 300px;
    height: 195px;
    transition: transform 1s;
    transform-style: preserve-3d;
    box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2);
  }

  .scene {
    width: 300px;
    left: 10%;
  }

  /* ---------CARD CONTROLLER STYLE-------------- */

  .card-controller {
    width: 300px;
  }

}
</style>
