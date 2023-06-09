<template>
  <!-- If user is logged in -->
  <div class="contain" v-if="store.state.user">
    <!-- Display summary of user account -->
    <div class="account-info">
      <span class="account-info-header">
        <u>Account Info</u>
      </span>
      <br />
      <br />
      <span>
        Email:
        <p>{{ this.email }}</p>
      </span>
      <span v-if="this.displayName">
        Name:
        <p>{{ this.displayName }}</p>
      </span>
      <span>
        Created:
        <p>{{ this.accountCreationDate }}</p>
      </span>
      <span>
        Topics:
        <p>{{ subjectsLength }}</p>
      </span>
      <br />
      <div class="delete-box">
        <img
          src="\edit-icon.png"
          class="account-edit-icon"
          @click.prevent="this.toggleEditAccountMode()"
        />
        <div
          v-if="editAccountMode"
          class="delete-account"
          @click="this.deleteAccount()"
        >
          Delete Account
        </div>
      </div>
    </div>
    <!-- Used to display and delete current user's subjects -->
    <div class="subject-editor">
      <h1>
        Topics
        <img
          src="\edit-icon.png"
          class="subject-edit-icon"
          @click.prevent="this.toggleEditMode()"
        />
      </h1>
      <hr />
      <div v-for="(item, key) in subjects" class="subject-list">
        <p>
          {{ item }}
          <img
            src="\cancel-icon.png"
            class="delete-subject-icon"
            style="width: 25px;"
            v-if="this.editMode"
            @click.prevent="this.deleteSubject(item)"
          />
        </p>
      </div>
    </div>
  </div>
  <!-- Landing page for non-user  -->
  <div v-else class="welcome-page">
    <h2 class="welcome-title">Recollect</h2>
    <div class="welcome-message">A Key Tool For Your Learning Journey</div>
    <div class="scene scene--card">
      <div class="welcome-card" v-bind:class="{ flipme: cardOne == 'flipped' }">
        <div class="welcome-card__face welcome-card__face--front">
          <div class="front-details">
            <span class="first-word">{{ animationWord }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// import { getDatabase, set, ref } from '../firebase/firebase'
import * as dotenv from "dotenv"
import { RouterLink } from 'vue-router'
import { Store, useStore } from 'vuex'
import store from '../store'

export default {
  data() {
    return {
      store: useStore(),
      user: '',
      displayName: '',
      email: '',
      accountCreationDate: '',
      cardOne: 'start',
      subjects: store.state.subjects,
      editMode: false,
      editAccountMode: false,
      animationWord: '',
    }
  },
  computed: {
    subjectsLength: () => {
      const store = useStore()
      const newArr = store.state.subjects
      return newArr.length
    },
    subjectArr() {
      if (this.store.state.subjects) {
        return this.store.state.stubjects
      } else {
        return 'No topics yet.'
      }
    },
  },
  methods: {
    deleteSubject(e) {
      let confirmDelete = confirm(
        `Are you sure you want to delete the topic ${e} and all of it's cards? `,
      )

      if (confirmDelete) {
        this.store.dispatch('deleteSubject', e)
        let arr = []

        for (let value of Object.values(this.subjects)) {
          if (value !== e) {
            arr.push(value)
          }
        }

        this.subjects = arr
      }
    },
    toggleEditMode() {
      this.editMode = !this.editMode
    },
    toggleEditAccountMode() {
      this.editAccountMode = !this.editAccountMode
    },
    welcomeAnimation() {
      setTimeout(() => {
        this.animationWord = 'Write'

        setTimeout(() => {
          this.animationWord = 'Study'

          setTimeout(() => {
            this.animationWord = 'Memorize'

            setTimeout(() => {
              this.animationWord = 'Learn'
            }, 1000)
          }, 1000)
        }, 1000)
      }, 1000)
    },
    getAccountInfo() {
      this.email = this.store.state.user.email
      this.displayName = this.store.state.user.displayName
      this.user = this.store.state.user
      const created = this.user.metadata.creationTime
      this.accountCreationDate = created.slice(0, -13)
      this.loadSubjects()
    },
    deleteAccount() {
      let confirmDelete = confirm(
        'Are you sure you want to delete your account?',
      )

      if (confirmDelete) {
        const db = dotenv.getDatabase()
        const userID = this.store.state.user.uid
        const path = `cards/${userID}`
        const details = { db, path, userID }
        this.store.dispatch('deleteUser', details)
      }
    },
    loadSubjects() {
      this.store.dispatch('loadSubjects')
      if (this.store.state.subjects) {
        this.subjects = this.store.state.subjects
      }
    },
  },
  mounted() {
    if (this.store.state.user) {
      this.getAccountInfo()
    } else {
      this.welcomeAnimation()
    }
  },
}
</script>

<style scoped>
.account-info {
  position: fixed;
  width: 500px;
  height: 350px;
  margin-top: 25px;
  padding: 5px;
  left: 22%;
  font-family: 'Marhey', cursive;
  font-size: 20px;
  font-weight: bolder;
  top: 25%;
  border-radius: 5%;
  /* background-color: #0bd692; */
  background-color: #107a57;
  box-shadow: 0 14px 28px rgba(0, 0, 0, 0.25), 0 10px 10px rgba(0, 0, 0, 0.22);
  padding: 35px;
  margin: 20px;
  overflow-y: scroll;
}

.account-info-header {
  font-size: 25px;
}

.account-info p {
  font-size: 20px;
  font-family: 'Marhey', cursive;
  color: rgb(67, 67, 67);
  text-indent: 25px;
}

.account-info span {
  /* color: rgb(0, 0, 0); */
  font-weight: 800;
  font-family: 'Marhey', cursive;
}

.account-title {
  left: 30%;
  margin-bottom: 25px;
}

.delete-box {
  height: 50px;
  justify-content: space-between;
  width: 300px;
}

.account-edit-icon {
  height: 20px;
  justify-content: flex-end;
  top: 25%;
}

.delete-account {
  position: absolute;
  font-weight: 800;
  color: rgb(134, 1, 1);
  width: fit-content;
  height: fit-content;
  left: 50%;
  top: 20%;
}

.delete-account:hover {
  cursor: pointer;
  text-decoration: underline;
}

/* Welcome Landing */
.welcome-page {
  height: 80vh !important;
  margin-top: 5%;
  text-align: center;
}

.welcome-title {
  padding: 5px;
  font-size: 60px;
  height: 100px;
  font-family: 'Rubik Vinyl', cursive;
  color: #107a57;
}

.welcome-message {
  font-size: 30px;
  font-family: 'Short Stack', cursive;
  color: #107a57;
}

/* Welcome Card View */
/* Card Preview */
.welcome-page .scene {
  position: absolute;
  width: 450px;
  height: 260px;
  perspective: 600px;
  justify-content: center;
  align-items: center;
  background-color: rgb(190, 190, 190);
  font-family: 'Merienda One', cursive;
  top: 40%;
  left: 32%;
}

.welcome-card {
  width: 100%;
  height: 100%;
  transition: transform 1s;
  transform-style: preserve-3d;
  position: fixed;
  box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2);
}

.welcome-card__face {
  width: 100%;
  height: 100%;
  text-align: center;
  font-weight: bold;
  backface-visibility: hidden;
  overflow-y: scroll;
  padding: 15px;
  font-size: 30px;
  justify-content: center;
  padding-top: 25%;
}

.flipme {
  transform: rotateY(180deg);
}

/* Subject Editor */
.subject-editor {
  position: fixed;
  width: 300px;
  height: 350px;
  margin-top: 25px;
  padding: 5%;
  left: 65%;
  font-family: 'Marhey', cursive;
  font-size: 15px;
  font-weight: bolder;
  top: 25%;
  border-radius: 5%;
  /* background-color: #0bd692; */
  background-color: #107a57;
  box-shadow: 0 14px 28px rgba(0, 0, 0, 0.25), 0 10px 10px rgba(0, 0, 0, 0.22);
  padding: 35px;
  margin: 20px;
  overflow-y: scroll;
}

.subject-list {
  font-size: 25px;
  margin-top: 15px;
}

.subject-edit-icon {
  position: absolute;
  height: 20px;
  top: 20%;
  left: 90%;
}

.subject-edit-icon:hover,
.account-edit-icon,
.delete-subject-icon:hover {
  cursor: pointer;
}

.subject-edit-icon:active,
.account-edit-icon,
.delete-subject-icon:active {
  transform: scale(0.9);
}

hr {
  background-color: #d2d8d6;
  border: none;
  height: 1px;
}

.delete-subject-icon {
  position: absolute;
  top: 5%;
  margin-left: 15px;
}

@media (max-width: 900px) and (min-width: 350px) {
  .account-info {
    width: 270px;
    height: 250px;
    padding: 5px;
    left: 15%;
    font-size: 15px;
    top: 15%;
    padding: 15px;
    margin: 0px;
    overflow-y: scroll;
    overflow-x: hidden;
  }

  .account-info-header {
    font-size: 20px;
  }

  .account-info p {
    font-size: 15px;
  }
  .delete-account {
    position: relative;
    font-weight: 600;
    left: 25%;
    top: 5%;
  }

  /* Landing Page */
  .welcome-page {
    position: fixed;
    left: 10%;
    top: 10%;
    height: 60vh;
    text-align: center;
  }

  .welcome-title {
    font-size: 23px;
    text-align: center;
  }

  .welcome-message {
    font-size: 15px;
  }

  /* Welcome Card View */
  /* Card Preview */
  .welcome-page .scene {
    height: 200px;
    width: 300px;
    left: 1%;
    top: 30%;
  }

  .welcome-card__face {
    font-size: 15px;
    padding-top: 30%;
  }

  /* Subject Editor */
  .subject-editor {
    width: 270px;
    height: 200px;
    padding: 5px;
    left: 15%;
    font-size: 15px;
    top: 55%;
    padding: 17px;
    margin: 0px;
    overflow-y: scroll;
    overflow-x: hidden;
    padding-top: 5%;
  }

  .subject-list {
    font-size: 20px;
    margin-top: 15px;
  }

  .subject-edit-icon {
    position: absolute;
    height: 20px;
    /* top: 90%; */
    left: 90%;
  }

  hr {
    background-color: black;
    border: none;
    height: 1px;
  }
}
</style>
