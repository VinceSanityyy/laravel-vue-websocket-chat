<template>
    <div class="row">
      <div class="col-8">
          <div class="card card-default">
              <div class="card-header">
                  Messages
              </div>
              <div class="card-body p-0">
                  <ul class="list-unstyled" style="height: 300px; overflow-y:scroll" v-chat-scroll>
                      <li class="p-2" v-for="(message, index) in messages" :key="index">
                          <strong>{{message.user.name}}: </strong>
                          {{message.message}} 
                          <br>
                          <small> {{message.created_at |  moment("dddd, MMMM Do YYYY, h:mm:ss a")}}</small>
                       
                      </li>
                  </ul>
              </div>
              <input @keydown="typingEvent" @keyup.enter="sendMessage" v-model="newMessage" class="form-control" type="text" name="" id="" placeholder="Enter msg">
              
          </div>
          <br>
           <span class="text-muted" v-if="activeUser">{{activeUser.name}} is typing...</span>
            <button v-on:click="sendMessage" class="btn btn-success btn-block">Send Message</button>
          
      </div>
      
      <div class="col-4">
          <div class="card card-default">
              <div class="card-header">
                  Current Active Users
              </div>
              <div class="card-body">
                  <ul style="color: green;  font-weight: bold;">
                      <li  class="py-2" v-for="(user, index) in users" :key="index">
                          {{user.name}}
                         
                      </li>
                  </ul>
              </div>
          </div>
      </div>
    </div>
</template>

<script>
    export default {
    
        data(){
            return{
                messages:[],
                newMessage: '',
                users: [],
                activeUser: false,
            }
        },
        props:[
            'user'
        ],
         created() {
            this.fetchMessages()
            // console.log('Component mounted.')
            Echo.join('chat')
                .here(user => {
                  this.users = user
                })
                .joining(user=>{
                  this.users.push(user)
                })
                 .leaving(user=>{
                   this.users = this.users.filter(u=> u.id != user.id)
                 })
                .listen('MessageSent',(event)=>{
                this.messages.push(event.message)
                 })
                .listenForWhisper('typing',response =>{
                    //  this.activeUser = user
                    console.log('typing')
                    console.log(response)
                     setTimeout(() => {
                         this.activeUser = false
                     }, 3000);
                 })
        },
        methods:{
            fetchMessages(){
                     axios.get('messages').then(res=>{
                    this.messages = res.data
                })
            },
            sendMessage(){
                this.messages.push({
                    user: this.user,
                    message: this.newMessage
                })
                axios.post('messages',{message: this.newMessage})

                this.newMessage = ''
                this.fetchMessages();
            },
            typingEvent(){
                Echo.join('chat')
                    .whisper('typing',this.user)
            }
        }
       
    }
</script>
