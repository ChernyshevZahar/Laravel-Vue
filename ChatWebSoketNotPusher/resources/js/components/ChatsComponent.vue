Skip to content Search or jump to… Pull requests Issues Marketplace Explore
@ChernyshevZahar qirolab / Laravel-WebSockets-Chat-Example 1 46 31 Code Issues 2
Pull requests Actions Projects Wiki Security Insights
Laravel-WebSockets-Chat-Example/resources/js/components/ChatsComponent.vue
@hkp22 hkp22 real-time chat example Latest commit 31b1fe7 on 31 Dec 2018 History
1 contributor 114 lines (96 sloc) 3.26 KB

<template>
    <div class="row">
        <div class="col-8">
            <div class="card card-default">
                <div class="card-header">Messages</div>
                <div class="card-body p-0">
                    <ul
                        class="list-unstyled"
                        style="height:300px; overflow-y:scroll"
                        v-chat-scroll
                    >
                        <li
                            class="p-2"
                            v-for="(message, index) in messages"
                            :key="index"
                        >
                            <strong>{{ message.user.name }}</strong>
                            {{ message.message }}
                        </li>
                    </ul>
                </div>

                <input
                    @keydown="sendTypingEvent"
                    @keyup.enter="sendMessage"
                    v-model="newMessage"
                    type="text"
                    name="message"
                    placeholder="Enter your message..."
                    class="form-control"
                />
            </div>
            <span class="text-muted" v-if="activeUser"
                >{{ activeUser.name }} is typing...</span
            >
        </div>

        <div class="col-4">
            <div class="card card-default">
                <div class="card-header">Active Users</div>
                <div class="card-body">
                    <ul>
                        <li
                            class="py-2"
                            v-for="(user, index) in users"
                            :key="index"
                        >
                            {{ user.name }}
                        </li>
                    </ul>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    props: ["user"],
    data() {
        return {
            messages: [],
            newMessage: "",
            users: [],
            activeUser: false,
            typingTimer: false
        };
    },
    created() {
        this.fetchMessages();
        Echo.join("chat")
            .here(user => {
                this.users = user;
            })
            .joining(user => {
                this.users.push(user);
            })
            .leaving(user => {
                this.users = this.users.filter(u => u.id != user.id);
            })
            .listen("MessageSent", event => {
                this.messages.push(event.message);
            })
            .listenForWhisper("typing", user => {
                this.activeUser = user;
                if (this.typingTimer) {
                    clearTimeout(this.typingTimer);
                }
                this.typingTimer = setTimeout(() => {
                    this.activeUser = false;
                }, 3000);
            });
    },
    methods: {
        fetchMessages() {
            axios.get("messages").then(response => {
                this.messages = response.data;
            });
        },
        sendMessage() {
            this.messages.push({
                user: this.user,
                message: this.newMessage
            });
            axios.post("messages", { message: this.newMessage });
            this.newMessage = "";
        },
        sendTypingEvent() {
            Echo.join("chat").whisper("typing", this.user);
        }
    }
};
</script>
