<template>
  <div>
    <new-meeting-form @added="addNewMeeting($event)"></new-meeting-form>

    <span v-if="meetings.length == 0">
               Brak zaplanowanych spotkań.
           </span>
     <div :class="'alert alert-' + (this.isError ? 'error' : '')" v-if="message">{{ message }}</div>
    <h3 v-else>
      Zaplanowane zajęcia ({{ meetings.length }})
    </h3>

    <meetings-list :meetings="meetings"
                   :username="username"
                   @attend="addMeetingParticipant($event)"
                   @unattend="removeMeetingParticipant($event)"
                   @delete="deleteMeeting($event)"></meetings-list>
  </div>
</template>

<script>
    import NewMeetingForm from "./NewMeetingForm";
    import MeetingsList from "./MeetingsList";

    export default {
        components: {NewMeetingForm, MeetingsList},
        props: ['username', 'meetings'],
        data() {
            return {
                message: '',
                isError: false
            };
        },
        methods: {
            addNewMeeting(meeting) {
            
            this.$http.post('meetings', meeting)
                    .then((response) => {
                         this.meetings.push({id: response.body.id, name:meeting.name, description: meeting.description, participants: meeting.participants});
                         this.success('');
                    })
                    .catch(response => this.failure('Błąd przy rejestracji spotkania. Kod odpowiedzi: ' + response.status));
            },
            addMeetingParticipant(meeting) {
            	 this.$http.post(`meetings/${meeting.id}/${this.username}`)
                    .then(() => {
                         meeting.participants.push(this.username);
                         this.success('');
                    })
                    .catch(response => this.failure('Błąd przy zapisywaniu się do spotkania. Kod odpowiedzi: ' + response.status));
                
            },
            removeMeetingParticipant(meeting) {
            	 this.$http.delete(`meetings/${meeting.id}/${this.username}`)
                    .then(() => {
                          meeting.participants.splice(meeting.participants.indexOf(this.username), 1);
                         this.success('');
                    })
                    .catch(response => this.failure('Błąd przy wypisywaniu się ze spotkania. Kod odpowiedzi: ' + response.status));
               
            },
            deleteMeeting(meeting) {
            	this.$http.delete(`meetings/${meeting.id}`)
                    .then(() => {
                         this.meetings.splice(this.meetings.indexOf(meeting), 1);
                         this.success('');
                    })
                    .catch(response => this.failure('Błąd podczas usuwania spotkania. Kod odpowiedzi: ' + response.status));
                
            },
             success(message) {
                this.message = message;
                this.isError = false;
            },
            failure(message) {
                this.message = message;
                this.isError = true;
            },
        }
    }
</script>
