<style scoped>
.chat-container {
    position: fixed;
    z-index: 998;
    bottom: 10px;
    right: 10px;
    border: 1px solid #aaa;
    box-shadow: 1px 2px 2px #aaa;
    background: #fff;
}

.chat-wrap {
}

.message-title {
    padding: 15px;
}

.user-serch {
    padding: 8px 4px;
}

/* 聊天窗口 */
.message-wrap {
    /* min-height: 300px; */
}

.message-items {
    background: #aaa;
    height: 320px;
    overflow: auto;
}

.message-item {
    word-wrap: break-word;
    word-break: break-all;
}

.message-text {
    padding: 10px;
    background: #fff;
    display: inline-block;
    margin: 5px;
    border-radius: 6px;
}

.message-submit-btn {
    margin-top: 2px;
}

.edit-message {
    padding: 4px 0;
}

.user {
    padding: 5px 7px;
}

.null-message {
    height: 460px;
}

.null-message .null-title {
    margin-top: 35%;
}

/* 显示私信窗口按钮 */
.show-chat-btn .show-chat-content {
    padding: 4px 10px;
    font-size: 1.2rem;
}
</style>

<template>
    <div v-if="uinfo || $store.state.chatComponent">
        <Modal v-model="$store.state.showChat" class="chat-container" width="700" footer-hide>
            <Row class="chat-wrap">
                <Row class="chat-main">
                    <Col :lg="6">
                    <div class="chat-user">
                        <div class="user-serch">
                            <Input placeholder="Enter text">
                            <Icon type="ios-search" slot="suffix" />
                            </Input>
                        </div>
                        <ul class="user-list cp-all bg-hv-all">
                            <li v-for="(user, index) in allList.user" :key="index" @click="userSelected(user)" v-if="uinfo && user.id!=uinfo.id" :class="{'bg-act': $store.state.form.to_id == user.id}" class="user">
                                {{user.username}}
                            </li>
                        </ul>
                    </div>
                    </Col>
                    <!-- {{$store.state.allList.message}} -->
                    <Col v-if="userItem.username" class="message-wrap" :lg="18">
                    <Row class="message-title">
                        <Col :lg="16">
                        <span>{{userItem.username}}</span>
                        </Col>
                    </Row>
                    <div class="message-items">
                        <div v-for="(message, index) in $store.state.itemList.message" v-if="(message.from_id == uinfo.id && message.to_id == $store.state.form.to_id) ||
                             (message.from_id == $store.state.form.to_id && message.to_id == uinfo.id)" class="message-item" :class="{'tar': message.from_id == uinfo.id}">
                            <p class="message-text">{{message.text}}</p>
                        </div>
                    </div>
                    
                    <Form class="edit-message">
                        <Input type="textarea" @on-keydown="handlerMultiEnter" v-model.trim="$store.state.form.text" :autosize="{minRows: 4,maxRows: 4}" placeholder="按下Enter发送内容/ Ctrl+Enter换行" />
                        <div class="message-submit-btn tar">
                            <Button @click.native="$store.state.form.text = null" :disabled="!$store.state.form.text">重置</Button>
                            <Button @click.native="$store.dispatch('sendMessage')" 
                            :disabled="!$store.state.form.text"
                            type="primary" style="margin-left: 6px">发送</Button>
                        </div>
                    </Form>
                    </Col>
                    <Col v-else :lg="18" class="null-message">
                    <div class="null-title">
                        <h2 class="tac">
                            暂无对话
                        </h2>
                    </div>
                    </Col>
                </Row>
            </Row>
        </Modal>
        <div @click="$store.commit('showChat')" v-if="!$store.state.showChat" class="show-chat-btn chat-container cp cl-hv">
            <p class="show-chat-content">
                <Icon type="md-mail" class="icon-mgr" size="25" />
                <span class="show-chat-title">私信聊天</span>
            </p>
        </div>
    </div>
</template>

<script>
import GReadInfo from "../mixins/GReadInfo";

import session from "../lib/session";

export default {
    mixins: [GReadInfo],
    data() {
        return {
            userItem: {},
            uinfo: session.uinfo(),
            messageTimer: null,
            splitLeft: 0.2,
            splitRight: 0.8
        };
    },
    mounted() {
        this.gReadInfo("user");
    },
    methods: {
        userSelected(user) {
            this.$set(this.$store.state.form, "to_id", user.id);
            this.$set(this.userItem, "username", user.username);
            this.$store.dispatch("readMessage");
            if (this.messageTimer) {
                clearInterval(this.messageTimer);
            }
            this.messageTimer = setInterval(() => {
                this.$store.dispatch("readMessage");
            }, 5000);
        },
        handlerMultiEnter(e) {
            let code = e.keyCode;
            let ctrl = e.ctrlKey;
            let shift = e.shiftKey;
            let alt = e.altKey;
            if (code == "13" && ctrl && !shift && !alt) {
                e.target.value += "\n";
            }
            if (code == "13" && !ctrl && !shift && !alt) {
                e.preventDefault();
                this.$store.dispatch("sendMessage");
                if (this.messageTimer) {
                    clearInterval(this.messageTimer);
                }
                this.messageTimer = setInterval(() => {
                    this.$store.dispatch("readMessage");
                }, 5000);
            }
        }
    }
};
</script>

