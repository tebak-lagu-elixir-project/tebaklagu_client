<template>
  <!-- <div id="Lobby">
    <h2>Welcome! Start by either creating a room or join a game!</h2>
    <hr/>
    <div class="row d-flex justify-content-center">
        <div class="col-5 p-2" style="background-color:#778beb">
          <div class="CreateRoomBox">
            <h2>Create a room</h2>
            <button type="button" class="btn btn-success">+ Create a room</button>
          </div>
          <div class="verticalLine"></div>
        </div>
        <div class="col-5 p-2" style="background-color:#f7d794">
          <h2>Currently available room: </h2>
          <div class="container">
              <div class="column">
                <a href="#">Game Room #0</a>
                <p>Dummy Room 1 #36</p>
                <p>Dummy Room 2 #37</p>
                <p>Dummy Room 3 #38</p>
                <p>Dummy Room 4 #39</p>
              </div>
            </div>
        </div>
    </div>
  </div> -->

    <div id="Lobby" class="container">
      <!-- Create Room -->
      <div class="form-container createRoom-container">
        <form>
          <h1>Create a Room :</h1>
          <!-- modal button Create Room -->
        <button type="button" class="btn btn-primary" data-toggle="modal" data-target="#ModalCreate" data-whatever="newtask" data-dismiss="modal" aria-label="close">create</button>
        <div class="modal fade" id="ModalCreate" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                <div class="modal-header">
                    <h1 class="modal-title" id="exampleModalLabel">Welcome, Game Master</h1>
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                    <span aria-hidden="true">&times;</span>
                    </button>
                </div>
                <div class="modal-body">
                    <form v-on:submit.prevent="createRoom">
                      <h2>Choose your genre !</h2>
                      <label for="roomName">Your room name:</label>
                      <input id="roomName" type="text" placeholder="nick name .." v-model="roomName"><br>
                      <select name="genre" id="genre" v-model="selectedGenre">
                        <option value="" disabled>Genre</option>
                        <option value="7644890062">Edm</option>
                        <option value="7644907862">Pop</option>
                        <option value="7644900062">Rock</option>
                        <option value="7644761062">Anime</option>
                        <option value="7644919622">Indonesia</option>
                      </select>
                    </form>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                    <button type="button" class="btn btn-primary" data-dismiss="modal" v-on:click.prevent="createRoom()">create</button>
                </div>
                </div>
            </div>
        </div>
        </form>
      </div>
      <!-- List Available Room -->
      <div class="overlay-container">
        <div class="overlay">
          <div class="overlay-panel overlay-right ">
          <h1>Currently available room</h1>
            <ul>
              <div v-for="room in allRooms" :key="room.id">
                <li><router-link :to="`/gameroom/${room.id}`" @click="joinRoom(room.id)">Game room {{room.name}}</router-link></li>
              </div>
            </ul>
            <img src="../assets/LunasMusic.gif" style="width: 50%;" >
          </div>
        </div>
      </div>
  </div>

</template>

<script>
import axios from 'axios'
import io from 'socket.io-client'
const socket = io.connect('https://shrouded-forest-27107.herokuapp.com')

export default {
  name: 'Lobby',
  data () {
    return {
      roomName: '',
      selectedGenre: '',
      allRooms: []
    }
  },
  methods: {
    joinRoom (roomId) {
      this.socket.emit('addPlayer', {
        roomId: roomId,
        newPlayer: this.currentUserName
      })
      io.on('addedPlayer', (room) => {
        this.$store.commit('set_joinedRoomData', room)
      })
    },
    createRoom () {
      console.log('creating room')
      axios({
        method: 'GET',
        url: `https://shrouded-forest-27107.herokuapp.com/songs/${this.selectedGenre}`
      })
        .then(result => {
          this.socket.emit('createRoom', {
            name: this.roomName,
            songs: result.data.songs,
            players: this.currentUserName
          })
        })
        .catch(error => {
          console.log(error)
        })
    }
  },
  computed: {
    roomNumber () {
      return this.$store.state.roomNumber
    },
    currentUserName () {
      return this.$store.state.currentUserName
    },
    socket () {
      return this.$store.state.socket
    }
  },
  created () {
    // socket.emit('showRooms')
    this.$store.commit('set_socket', socket)
    socket.on('createdRoom', (room) => {
      console.log('Ini adalah created room', room)
      this.$store.commit('set_joinedRoomData', room)
      this.$router.push(`/gameroom/${room.id}`)
    })
    socket.emit('getRooms')
    socket.on('showRooms', (rooms) => {
      console.log('LOBBY DARI SHOWROOMS', rooms)
      this.$store.commit('set_allRooms', rooms)
      this.allRooms = rooms
    })
  }
}
</script>

<style>

.verticalLine{
    width:5px;
    height:auto;
    background-color:#ffc048;
    border-radius: 5px;
}

</style>
