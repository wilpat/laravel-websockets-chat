<template>
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card card-default">
                <div class="card-header">Messages</div>

                <div class="card-body p-0">
                    <ul class="list-unstyled" style="height:300px; overflow-y:scroll;">
                        <li class="p-2" v-for="(message, index) in messages" :key="index">
                            <strong>{{ message.user.name.toUpperCase() }}: </strong>
                            {{message.message}}
                        </li>
                    </ul>
                </div>

                <input @keyup.enter="sendMessage" v-model = "newMessage" type="text" name = "message" placeholder="Enter your message" class="form-control">
            </div>

            <span class="text-muted">User is typing...</span>
        </div>

        <div class="col-md-4">
            <div class="card card-default">
                <div class="card-header">Active Users</div>
                <div class="card-body">
                    
                    <ul>
                        <li class="py-2" v-for ="user in users">{{ check( user )}}</li>
                    </ul>

                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {

        props:['user'],

        data(){
            return {
                messages : [],
                newMessage : '',
                users: []
            }
        },

        created(){
            this.fetchMessages();

            Echo.join('chat')
                .here(activeUsers =>{
                    this.users = activeUsers;
                })
                .joining(newUser =>{
                    this.users.push(newUser);
                }).leaving(leavingUser=>{
                    let id = this.users.indexOf(leavingUser);
                    this.users.splice(id,1)
                })
                .listen('MessageEvent', (event) =>{
                    this.messages.push(event.message)
                })
        },

        methods:{
            fetchMessages(){
                axios.get('messages')
                     .then(response =>{
                        this.messages = response.data
                     })
            },

            sendMessage(){
                this.messages.push({
                    user : this.user,
                    message : this.newMessage
                });

                axios.post('message', {message : this.newMessage})
                    .then(response =>{
                        // console.log('sent');
                    })
                    .catch(error =>{
                        // console.log('oops!');
                    })
                this.newMessage = '';
            },

            capitalize(string) {
                return string.charAt(0).toUpperCase() + string.slice(1);
            },

            check(person){
                return person.id == this.user.id ? this.capitalize('You') : this.capitalize(person.name) ;
            }
        }
    }
</script>
