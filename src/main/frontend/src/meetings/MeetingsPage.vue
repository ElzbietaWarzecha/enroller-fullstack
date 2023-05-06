<template>
  <div>
    <NewMeetingForm @added="addNewMeeting($event)"></NewMeetingForm>
    <span v-if="meetings.length == 0">Brak zaplanowanych spotkań.</span>
    <h3 v-else>Zaplanowane zajęcia ({{ meetings.length }})</h3>
    <MeetingsList :meetings="meetings"
                  :username="username"
                  @attend="addMeetingParticipant($event)"
                  @unattend="removeMeetingParticipant($event)"
                  @delete="deleteMeeting($event)"></MeetingsList>
  </div>
</template>

<script>
import NewMeetingForm from "./NewMeetingForm";
import MeetingsList from "./MeetingsList";
import axios from "axios";

export default {
  components: {NewMeetingForm, MeetingsList},
  props: {username: String},
  data() {
    return {
      meetings: [],
      message: '',
      isError: false,
    };
  },
  created() {
    axios.get('/api/meetings')
          .then(response => (
            this.meetings = (response.data)
          ))
          .catch(error => console.log(error))
    },
  methods: {
    addNewMeeting(meeting) {
      this.meetings.push(meeting);
        axios.post('/api/meetings', meeting)
          .then(() => {
            this.success('Spotkanie zostało dodane.');
          })
          .catch(error => console.log(error))
    },
    addMeetingParticipant(meeting) {
      meeting.participants.push(this.username);
    },
    removeMeetingParticipant(meeting) {
      meeting.participants.splice(meeting.participants.indexOf(this.username), 1);
    },
    deleteMeeting(meeting) {
      axios.delete('/api/meetings/' + meeting.id)
          .then(() => {
            console.log('Spotkanie zostało usunięte.');
          })
          .catch(error => console.log(error))
          this.meetings.splice(this.meetings.indexOf(meeting), 1);
    },
    success(message) {
      this.message = message;
      this.isError = false;
    },
    failure(message) {
      this.message = message;
      this.isError = true;
    },
    clearMessage() {
      this.message = undefined;
    },
  }
}
async function returnMeetingByTitle(meeting){
        let config = {
        params: {
          title: meeting.title
        }
      }
    let response = await axios.get('/api/meetings', config)
          .then(() => {
            foundMeeting = response.data;
            console.log(response.data);
            return response.data;
          })
          .catch(error => {
            console.log('Błąd!!!');
          });
          
        }  
       
</script>
