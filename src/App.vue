<template>
  <div class="container">
      <div class="form">
          <div class="sidebar">
              <div class="sbarData">
                  <h1>Log Job Via Email</h1>
                  <p>
                      Use this template to save your email settings with specific senders
                      subjects and custom keys.
                  </p>
                  <p>Email to: stgautomation@gmail.com</p>
                  <p>Please follow these guidelines before you start:</p>
                  <p>
                  Senders: Add only the emails you'll use to contact the automation
                  team.
                  </p>
                  <p>
                  Email Subjects: The subject must be unique include the 5-digit key
                  displayed below the subject when emailing.
                  </p>
                  <p>
                  Custom Keys: Default keys are provided. You can edit and/or add them
                  to the email body with the "+" button. You can remove keys from email
                  body using the "-" button.
                  </p>
              </div>
          </div>
          <div class="content">
              <!-- toggle button  -->
              <button @click="togglePage" class="toggle-button"> View History</button>

              <!-- form -->
              <div v-if="showForm" class="form-container">
                  <h2>Entry Form</h2>
                  <form @submit.prevent="handleSave">
                          <p>Sender *</p>
                          <InputText type="email" v-model="senderInput" id="inp1" placeholder="Enter sender email"  @keydown.enter.prevent="addSender"    /> 

                          <!-- Error message for invalid email -->
                          <p v-if="emailError" class="error-message">{{ emailError }}</p>

                          <!-- Display added senders as tags -->
                          <div v-if="senders.length > 0" class="sender-list">
                              <span 
                                  v-for="(sender, index) in senders" 
                                  :key="index" 
                                  class="sender-item"
                                  >
                                  {{ sender }}
                              <span 
                                  class="remove-sender" 
                                  @click="removeSender(index)"
                                  >x</span>
                              </span>
                          </div>

                          <p>Email Subject *</p>
                          <InputText type="text" v-model="value.inp2" id="inp2"/>

                          <p>Custom Keys *</p>
                          <div class="keys">
                              <div v-for="(key, index) in customKeys" :key="index" >
                                  <div v-if="key.isEditing" class="keyData">
                                      <input v-model="key.text" class="keyInp" />
                                      <button id="keyInpBtn" type="button" @click="saveEditedKey(index)">✔️</button>
                                  </div>
                                  <span v-else class="keybtn">
                                      <span class="key-text" @click="addTagToEmailBody(key)">{{ key.text }}</span>
                                      <button type="button" @click="addTagToEmailBody(key)">+</button>
                                      <button type="button" @click="removeTagFromEmailBody(key)">-</button>
                                      <span class="edit-icon" @click="editKey(index)">✏️</span> <!-- Pencil Icon for Editing -->
                                  </span>
                              </div>
                          </div>

                          <p> Email Body</p>
                          <Textarea id="textarea" v-model="value.textarea" rows="5" cols="30"   @click="setCursorPosition" /> 
                  
                          <div class="buttons">
                              <Toast />
                              <Button label="Reset"  severity="info" type='button' @click="handleCancel" />
                              <Button label="Save" severity="success" @click="showSuccess()" type="submit" />
                          </div>
                      </form>
                  </div>
                  <div v-else class="historyPage">
                      <h1>History</h1> 
                      <DataTable :value="info" paginator :rows="10" :rowsPerPageOptions="[5, 10, 20, 50]" tableStyle="min-width: 50rem">
                          <Column field="code" header="Code"></Column>
                          <Column field="Email" header="Email"></Column>
                          <Column field="Department" header="Department"></Column>
                      </DataTable>
                  </div>
          </div>
      </div>
  </div>
</template>

<script >

import  { ref  } from 'vue';
const data = ref();

export default{
  data(){
    return{
      senderInput: "", // User input email
      emailError: "", // Error message for email validation
      showForm: true,
      value:{
        inp2:'',
        textarea:'- Job Detail: Web Developer \n- Experience: 2 Years', 
      },
      senders: [],
      tagsHistory: [], // Stack to track the history of inserted tags

      // Array for custom keys
      customKeys: [
        { text: "Hello!!!", inserted: false, isEditing: false },
        { text: "How Can I Help You!", inserted: false, isEditing: false },
        { text: "Thanks", inserted: false, isEditing: false },
      ], 
      info:[
        {code : '123' , Email : 'xyz@gmail.com' , Department : 'WEB'},
        {code : '456' , Email : 'abc@gmail.com' , Department : 'Python'},
        {code : '789' , Email : 'xyz@gmail.com' , Department : 'WEB'},
        {code : '123' , Email : 'abc@gmail.com' , Department : 'Python'},
        {code : '456' , Email : 'xyz@gmail.com' , Department : 'WEB'},
        {code : '789' , Email : 'abc@gmail.com' , Department : 'Python'},
        {code : '123' , Email : 'xyz@gmail.com' , Department : 'WEB'},
        {code : '456' , Email : 'abc@gmail.com' , Department : 'Python'},
        {code : '789' , Email : 'xyz@gmail.com' , Department : 'WEB'},
        {code : '123' , Email : 'abc@gmail.com' , Department : 'Python'},
        {code : '456' , Email : 'xyz@gmail.com' , Department : 'WEB'},
        {code : '789' , Email : 'abc@gmail.com' , Department : 'Python'},
      ],
    };
  },
  methods: {
    togglePage() {
      this.showForm = !this.showForm;
    },
    validateEmail(email) {
      const emailPattern = /^[a-zA-Z0-9._%+-@]+\.com$/;
      return emailPattern.test(email);
    },
    addSender() {
      const email = this.senderInput.trim();
      if (email === "") {
        this.emailError = "Email cannot be empty.";
        return;
      }
      if (!this.validateEmail(email)) {
        this.emailError = "Please enter a valid email ending with .com.";
        return;
      }
      if (this.senders.includes(email)) {
        this.emailError = "This email has already been added.";
        return;
      }
      this.senders.push(email);
      this.senderInput = "";
      this.emailError = ""; // Clear error after successful addition
    },
    removeSender(index) {
      this.senders.splice(index, 1);
    },

    // Add tag to email body
    addTagToEmailBody(key) {
      const tag = `\n- ${key.text}`;
      this.value.textarea += tag; // Append the tag to the textarea
      // Record the tag in the tagsHistory for tracking
      this.tagsHistory.push({key,tag});
      key.inserted = true; // Mark as inserted
    },

    removeTagFromEmailBody(key) {
      const historyIndex = this.tagsHistory.findIndex(history => history.key === key);
      if (historyIndex !== -1) {
        const { tag } = this.tagsHistory[historyIndex];
        const lastIndex = this.value.textarea.lastIndexOf(tag);
        if (lastIndex !== -1) {
          // Remove the tag from the email body
          this.value.textarea =
          this.value.textarea.substring(0, lastIndex) +
          this.value.textarea.substring(lastIndex + tag.length);
        }
        // Remove the tag from history
        this.tagsHistory.splice(historyIndex, 1);
      }
      key.inserted = false; // Mark the key as not inserted
    },
    editKey(index) {
      // Enable editing mode and save the old text for replacement if needed
      const key = this.customKeys[index];
      key.isEditing = true;
      key.oldText = key.text; // Store old text for email body replacement
    },
    // Save the edited key and update the email body
    saveEditedKey(index) {
      const key = this.customKeys[index];
      const oldText = key.oldText; // Store the old text for replacement
      const newText = key.text; // Use the updated text
      
      // If the key was already inserted into the email body, replace it with the new text
      if (key.inserted) {
        // Replace old tag with the new edited tag in the email body
        const oldTag = `\n- ${oldText}`;
        const newTag = `\n- ${newText}`;
        this.value.textarea = this.value.textarea.replace(oldTag, newTag);
        // Update the tag history to reflect the new tag
        const historyIndex = this.tagsHistory.findIndex(history => history.key === key);
        if (historyIndex !== -1) {
          this.tagsHistory[historyIndex].tag = newTag;
        }
      }
      // Exit editing mode
      key.isEditing = false;
      key.oldText = null; // Clear the old text
      key.inserted = true; // Ensure the tag is marked as inserted
    },
    cancelEdit(index) {
      this.customKeys[index].isEditing = false;
    },
    handleCancel() {
      this.value.inp2 = "",
      this.value.textarea = "",
      this.senderInput = "";
      this.senders = [];
      this.$toast.add({ severity: 'info', 
      summary: 'Form Reset', 
      detail: 'Clear Form Data!', 
      life: 3000 });
    },
    handleSave() {
      this.value.inp2 = "",
      this.value.textarea = "",
      this.senderInput = "";
      this.senders = [];
    },
    showSuccess() {
      this.$toast.add({ severity: 'success', 
      summary: 'Form Submitted', 
      detail: 'Thanks For Your Details', 
      life: 3000 });
    },
  },
};
</script>

<style scoped>
.container {
  height: 90vh;
  width: 100%;
  font-family: sans-serif;
 
}
.form {
  height: 95vh;
  width: 95vw;
  margin: auto;
  display: flex;
}
.sidebar {
  height: 95vh;
  width: 30vw;
  background-color: cadetblue;
  color: white;
  margin: auto;
  border-top-left-radius: 10px;
  border-bottom-left-radius: 10px;
}
.sbarData{
  height: 80vh;
  width: 25vw;
  margin: 8vh 0vw 0vw 1.6vw;
  font-size: 2.5vh;
  border-bottom: 5px solid dodgerblue;
  border-bottom-left-radius: 5px;
  border-bottom-right-radius: 5px;
}
.sbarData h1,p{
  padding: 0vw 0vw 0vw 0.5vw;
}
.content {
  height: 95vh;
  width: 70vw;
  margin: auto;
  padding: 0vw 0vw 0vw 2vw;
  border: 1px solid black;
  border-top-right-radius: 10px;
  border-bottom-right-radius: 10px;
}
.toggle-button{
  height: 5vh;
  width: 10vw;
  background-color: dodgerblue;
  color: white;
  cursor: pointer;
  border: none;
  border-radius: 10px;
  margin: 3vh 0 0 45vw;
  font-size: 1vw;
}
.toggle-button:hover{
  background-color: rgb(5, 118, 231);
}
.form-container h2{
  font-size: 1.6vw;
  font-weight: 700;  
}

.input p{
  font-size: 1.2vw;
  font-weight: 600;
}
#inp1{
  height: 5vh;
  width: 85%;
  
}
#inp2{
  height: 5vh;
  width: 85%;
}
.keys{
  height: 10vh;
  border: 1px solid rgb(197, 195, 195);
  border-radius: 10px;
  color: #fff;

}
#textarea{
  height: 13vh;
  width: 85%;
  font-size: 1.1vw;
}
.buttons{
  height: 6vh;
  width: 20vw;
  display: flex;
  gap: 0.5vw;
  margin: 0.4vw 0vw 0vw 36vw;
}
.buttons button{
  height: 5vh;
  width: 9vw;
  border: none;
  background-color: cadetblue;
  color: #fff;
  border-radius: 10px;
  cursor: pointer;
  font-size: 1.1vw;
}

.sender-item {
  background-color: cadetblue;
  color: white;
  padding: 0.1vw 1vw;
  border-radius: 5px;
  font-size: 0.9vw;
  display: inline-block;
  margin-right: 10px;
  margin-top: 0.2vw;
}
.remove-sender {
  margin-left: 0.5vw;
  cursor: pointer;
  color: #dd1c1c;
}
.error-message {
  color: #fc1a2d;
}
.keys{
  height: 10vh;
  width: 85%;
  display: flex;
  justify-content: center;
  align-items: center;
  justify-content: space-evenly;
}
.keybtn{ 
  height: 6.5vh;
  width: 17vw;
  display: flex;
}
.key-text{
  height: 5vh;
  width: 9.5vw;
  margin: auto;
  font-size: 0.9vw;
  border: 1px solid rgb(70, 69, 69);
  padding: 0.7vw 0vw 0vw 0.3vw;
  border-radius: 5px;
  color: black;
  cursor: pointer;
}
.edit-icon{
  height: 3.5vh;
  width: 1.8vw;
  margin: auto;
  font-size: 0.8vw;
  border: 2px solid rgb(65, 65, 65);
  padding: 0.2vw 0vw 0vw 0.3vw;
  border-radius: 5px;
  cursor: pointer;
}
.keybtn button{
  width: 1.7vw;
  height: 3.5vh;
  margin: auto;
  border-radius: 5px;
  cursor: pointer;
  font-size: 1.2vw;
  font-weight: 500;
}
.keyInp{
  height: 5.2vh;
  width: 14vw;
  font-size: 1.1vw;
  border-radius: 5px;
  padding: 0.3vw 0vw 0vw 0vw;
}
#keyInpBtn{ 
  height: 4.8vh;
  width: 3vw;
  margin: 0.2vw;
  border-radius: 5px;
  font-size: 1.2vw;
}
.historyPage{
  height: 80vh;
  width: 60vw;
  font-size: 1vw;
}

@media ( max-width :700px) {
  .sidebar{
      display: none;
  }
  
}

</style> 


 
