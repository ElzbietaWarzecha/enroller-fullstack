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

// const AsyncComponent = () => ({
//   component: import ('./MeetingsList.vue'),
// })

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
    let dbData = [];
    let participantResponse = [];
    axios.get('/api/meetings')
          .then(response => (
            dbData = (response.data),
                dbData.forEach(element => {
                    axios.get('/api/meetings/' + element.id + '/participants')
                      .then(response => (
                        participantResponse = (response.data),
                        element.participants = [],
                        participantResponse.forEach(pr => {
                            element.participants.push(pr.login)
                          })
                        //element.participants = response.data
                      ))
                      .catch(error => console.log(error))
            }), 
            this.meetings = dbData   
              ))
          .catch(error => console.log(error));    
          $event(this.meetings);  
  },
  // mounted() {
  //         this.meetings.forEach(element => {
  //       axios.get('/api/meetings/' + element.id + '/participants')
  //       .then(response => (
  //         element.participants = (response.data)
  //           ))
  //           .catch(error => console.log(error));
  //     });
  //   },
  methods: {
    addNewMeeting(meeting) {
      this.meetings.push(meeting);
        axios.post('/api/meetings', meeting)
        .then(response => {
            console.log('Spotkanie zostało dodane.');
          })
          .catch(error => console.log(error))
    },
    addMeetingParticipant(meeting) {
      this.meetings.at(this.meetings.indexOf(meeting)).participants.push(this.username);
      axios.post('/api/meetings/' + meeting.id + '/participants', {login:this.username} )
        .then(() => {
          console.log('Uczestnik nie został dodany do spotkania');
        })
    },
    removeMeetingParticipant(meeting) {
      let currentMeeting = this.meetings.at(this.meetings.indexOf(meeting));
      currentMeeting.participants.splice(this.username);
      //meeting.participants.splice(meeting.participants.indexOf(this.username), 1);
      axios.delete('/api/meetings/' + currentMeeting.id + '/participants/' + this.username)
        .then(() => {
          console.log('Uczestnik nie został usunięty ze spotkania');
        })
    },
    deleteMeeting(meeting) {
      let currentMeeting = this.meetings.at(this.meetings.indexOf(meeting));
      this.meetings.splice(currentMeeting);
      //this.meetings.splice(this.meetings.indexOf(meeting), 1);
      axios.delete('/api/meetings/' + currentMeeting.id)
          .then(() => {
            console.log('Spotkanie zostało usunięte.');
          })
          .catch(error => console.log(error))
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
       
</script>
