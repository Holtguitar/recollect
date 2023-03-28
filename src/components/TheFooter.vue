<template>
  <div class="footer-bar">
    <div class="contact-section">
      Contact us with any issues or concerns @ doppio.web.designs@gmail.com
    </div>
    <div class="doppio-section">
      <div class="doppio-text">Powered by © Doppio Web Designs</div>
      <a 
        class="doppio-link" 
        href="https://brandon-holt-portfolio.netlify.app/"
        target="_blank"
      >
        <img src="/doppio-logo.png" class="doppio-logo"/>
      </a>
    </div>
  </div>
  </template>
  
  <script>
    import { useStore } from "vuex"
  
    export default {
      data(){
        return {
          store: useStore()
        }
      },
      methods: {
        handleSignOut(){
          this.store.dispatch("logout");
        }
      }
    }
  
  </script>
  
  <style>
    .footer-bar {
      position: fixed;
      width: 100vw;
      height: 70px;
      bottom: 0%;
      left: 0%;
      background-color: rgb(245, 245, 245);
      display: flex;
      font-family: 'Short Stack', cursive;
    }
  
    .doppio-section {
      font-size: 13px;
      height: 60px;
      
      left: 30%;
      text-align: center;
      display: flex;
      flex-direction: row;
      color: black;
    }

    .doppio-logo {
      height: 80%;
      transform: rotate(19deg);
      top: 10px;
    }

    .doppio-text {
      top: 30%;
    }

    .doppio-link {
      top: 0%;
      width: 70px;
      height: 100%;
      left: 5%;
    }

    .doppio-link:hover {
      cursor:pointer;
    }

    .contact-section {
      position: relative;
      /* background-color: red; */
      color: black;
      width: 600px;
      left: 10%;
      top: 10px;
    }

    @media (max-width: 900px) and (min-width: 380px){

      .footer-bar {
        height: 45px;
      }

      .doppio-section {
        left: 7%;
      }

      .doppio-link {
        left: 1%;
      }
    }  
  </style>