<template>
  <v-container>
    <v-row style="height: 500px;" justify="center">
      <v-col cols="12">
        <v-sheet class="pa-2 rounded-lg" elevation="12">

          <v-card elevation="4">
            <div class="pa-4 text-center">
              <h3>{{ selectedOption }} <v-icon color="green">mdi-account-badge-outline</v-icon></h3>
            </div>
          </v-card>

          <div class="chat-messages">
            <v-row v-for="message in messages" :key="message.id" :justify="message.type === 'A' ? 'start' : 'end'"
              no-gutters>

              <div class="pa-3 mt-3 rounded-xl" xs="12" :id="'chat-' + message.id" round="4" :style="{
                textAlign: message.type === 'A' ? 'start' : 'end',
                backgroundColor: message.type === 'A' ? '#FCF6F5FF' : '#FAEBEFFF',
                maxWidth: '50%',
                overflowWrap: 'break-word'
              }">

                <v-icon v-if="message.type === 'A'" color="green">mdi-account-badge-outline</v-icon>
                {{ message.text }}
                <v-icon v-if="message.type !== 'A'" color="green">mdi-chat</v-icon>

                <div style="font-size: 10px;" v-if="message.type === 'Q' && message.sending">Sending <v-icon
                    color="orange">mdi-message-text-fast</v-icon></div>
                <div style="font-size: 10px;" v-if="message.type === 'Q' && message.sent">Sent {{ message.time }}
                  <v-icon color="green">mdi-check-all</v-icon>
                </div>
              </div>
            </v-row>
          </div>

          <v-row class="pt-5" align="center">
            <v-col xs="10" sm="10" md="10" lg="11" xl="11"><v-text-field variant="outlined" label="Type here"
                hide-details v-model="newMessageText" required></v-text-field>
            </v-col>
            <v-col xs="2" sm="2" md="2" lg="1" xl="1">
              <v-btn @click="getData" icon="mdi-send" color="primary"></v-btn>
            </v-col>
          </v-row>
          <v-row class="pb-5">
            <v-col cols="6">
              <v-select v-model="selectedOption" variant="outlined" :items="options" hide-details required
                label="Choose the model"></v-select>
            </v-col>
          </v-row>


        </v-sheet>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>

import axios from 'axios';
import anime from 'animejs/lib/anime.es.js';

export default {
  data() {
    return {
      messages: [],
      options: ["Finetune AI Bot", "Base Model"],
      selectedOption: "Finetune AI Bot",
      newMessageText: '',
      polly: null,
    };
  },
  mounted() {

      this.polly = new AWS.Polly({
        region: 'us-east-1',
        accessKeyId: 'AKIAZZIJIA56NOTTOEVS',
        secretAccessKey: '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',
      });
      console.log(this.polly)
  
  },
  methods: {
    addMessage() {

      this.messages.push({
        id: this.messages.length + 1,
        text: this.newMessageText,
        type: "Q",
        sending: false,
        sent: false,
        time: 0
      });

      this.newMessageText = "";
    },
    async getData() {

      await this.addMessage();

      let element = document.getElementById("chat-" + this.messages.length)
      let chat_messages = document.getElementsByClassName("chat-messages")[0]
      let elementRect = element.getBoundingClientRect();
      let elementTop = elementRect.top;
      let elementBottom = elementRect.bottom;
      let chat_messagesRect = chat_messages.getBoundingClientRect();

      if (elementTop < chat_messagesRect.top || elementBottom > chat_messagesRect.bottom) {

        element.scrollIntoView({ behavior: 'smooth', block: 'center' });
      }

      anime({
        targets: element,
        scale: [0, 1],
        translateY: [100, 0],
        duration: 150,
        easing: 'easeOutQuad',
        autoplay: true
      });

      this.messages.at(this.messages.length - 1).sending = true

      // const response = await axios.get('https://jsonplaceholder.typicode.com/posts');

      setTimeout(() => {
        this.messages.at(this.messages.length - 1).sending = false
        this.messages.at(this.messages.length - 1).sent = true
        let date = new Date();
        this.messages.at(this.messages.length - 1).time = date.getHours() + ":" + date.getMinutes()
      }, 2000);

      const params = {
        OutputFormat: 'mp3',
        Text: "Test voice",
        VoiceId: 'Joanna'
      };

      const response = await this.polly.synthesizeSpeech(params).promise();
      const aContext = new AudioContext();

      const source = aContext.createBufferSource();
      source.buffer = await aContext.decodeAudioData(response.AudioStream.buffer);
      source.connect(aContext.destination);
      source.start();


    }
  }
};
</script>


<style>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400&display=swap');

* {
  font-family: 'Poppins', sans-serif;
}

.chat-messages {
  background-color: #9CC3D5FF;
  height: 60vh;
  overflow-y: scroll;
  padding: 10px;
}


.chat-messages::-webkit-scrollbar {
  width: 10px;
  background-color: #c9c9c9;
}

.chat-messages::-webkit-scrollbar-thumb {
  border-radius: 10px;
  background-color: #467fa0;
}
</style>
