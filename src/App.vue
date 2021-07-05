<template>
  <div id="app" class="base-app">
    {{ message }}

    <button @click="call">Call</button>

    <p v-if="waiting">...waiting</p>

    <template v-if="recievingCall">
      <h3>receiving call</h3>
      <button @click="ignoreCall">Ignore</button>

      <button @clcik="answerCall">Answer</button>
    </template>

    <template v-if="ignored">
      <p>Call Not Answer</p>
      <p>Reason: {{ ignored.reason }} </p>
    </template>

    <template v-if="answered">
      <p>... ongoing call</p>
    </template>

    <template v-if="callEnded">
      <p>
        call ended
      </p>
    </template>
  </div>
</template>

<script>
  import io from "socket.io-client";
  export default {
    name: "app",
    data() {
      return {
        socket: io("localhost:3000"),
        payload: null,
        waiting: false,
        ignored: null,
        recievingCall: false,
        caller: null,
        answered: false,
        callEnded: false,
        message: null
      }
    },
    mounted() {
      this.socket.emit("welcome", {
        name: "laji"
      })

      this.socket.on("user logged in", payload => {
        this.message = payload
      })


      this.socket.on("call-waiting", data => {
        this.message = data
        this.waiting = data.wait
      })

      this.socket.on("receiving-call", payload => {
        this.caller = payload.from
        this.recievingCall = true
      })

      this.socket.on("call-ignored", payload => {
        this.ignored = payload
      })

      this.socket.on("call-answered", payload => {
        this.answered = true
        this.message = `On going call with ${payload.caller}`
      })

      this.socket.on("call-ended", () => {
        this.callEnded = !this.callEnded
        this.onCallEnded(true)
      })

    },

    computed: {

      onCallEnded: function (bool) {
        if (bool) {
          return setTimeout(() => this.message = "Call ended", 3000)
        }
        this.callEnded = !this.callEnded
      },
    },

    methods: {

      call() {
        this.socket.emit("make-call", {
          user: "laji",
          id: "234"
        })
      },

      ignoreCall() {
        this.socket.emit("ignore-call", {
          answer: false,
          reason: "busy",
          user: {
            name: "reciever"
          }
        });
      },

      answerCall() {
        this.socket.emit("answer-call", {
          answer: true,
          user: {
            name: "reciever"
          }
        })
      },

      endCall() {
        this.socket.emit("end-call", )
      },

    }
  }
</script>

<style lang="scss">

</style>
