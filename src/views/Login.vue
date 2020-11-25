<template>
<div class="login-wrapper">
    <transition name="login">
        <div class="login" v-if="isShow">
            <div class="l">
                <h1>欢迎使用<br>北工商设备管理系统</h1>
            </div>
            <div class="r">
                <ul class="clear-fix">
                    <li
                        v-for="d in typeList"
                        :key="d.type"
                        @click="type = d.type"
                        :class="{ 'active': type === d.type }"
                    >{{ d.label }}</li>
                </ul>
                <div class="form">
                    <div v-show="type === 'sms'" class="sms-form">
                        <div class="input">
                            <el-input v-model="phone" type="text" placeholder="手机号"></el-input>
                        </div>
                        <div class="input">
                            <el-input
                                v-model="code"
                                type="text"
                                placeholder="验证码"
                                @keyup.native.enter="login"
                            ></el-input>
                            <el-button
                                type="primary"
                                :disabled="!phone || time !== 60"
                                @click="sendCode"
                            >{{ sendBtnLabel }}</el-button>
                        </div>
                    </div>
                    <div v-show="type === 'password'" class="password-form">
                        <div class="input">
                            <el-input v-model="username" placeholder="用户名"></el-input>
                            <!-- <input v-model="username" type="text" placeholder="用户名"> -->
                        </div>
                        <div class="input">
                            <el-input
                                v-model="password"
                                type="password"
                                placeholder="密码"
                                @keyup.native.enter="login"
                            ></el-input>
                            <!-- <input v-model="password" type="password" placeholder="密码"> -->
                        </div>
                    </div>
                    <!-- <button @click="login">登陆</button> -->
                    <el-button type="primary" @click="login">登陆</el-button>
                </div>
            </div>
        </div>
    </transition>
</div>
</template>

<script>
import axios from 'axios';
export default {
    data() {
        return {
            isShow: false,
            typeList: [{
                type: 'sms',
                label: '短信登陆'
            }, {
                type: 'password',
                label: '密码登陆'
            }],
            type: 'sms',
            phone: '',
            code: '',
            username: '',
            password: '',
            time: 60,
            sendBtnLabel: '发送验证码',
            timeId: null
        }
    },
    methods: {
        login() {
            const { type, phone, code, username, password } = this;
            let params = {
                type
            };
            if (type === 'sms') {
                params.phone = phone;
                params.code = code;
            } else {
                params.username = username;
                params.password = password;
            }
            axios.post('/equipment/login', params).then(({ data }) => {
                if (!data.code) {
                    this.$router.push('/main/overview');
                } else {

                    this.$message({
                        message: data.msg,
                        type: 'warning'
                    });

                    // alert(data.msg);
                }
            });
        },
        getCode() {
            axios.post('/equipment/code', {
                phone: this.phone
            }).then(({ data }) => {
                if (!data.code) {
                    this.$message.success('验证码已经发送到您的手机，请查收');
                }
            });
        },
        sendCode() {
            /* 先对手机号校验，以1开头，后面跟10位数字*/
            const { phone } = this;
            if (!phone) return this.$message.warning('请输入您的手机号');
            if (!/^1\d{10}$/.test(phone)) return this.$message.warning('请输入格式正确的手机号');
            // 通过手机号码校验后，调用接口，并开始倒计时
            this.getCode();
            this.time--;
            this.sendBtnLabel = `${this.time}秒后重发`;
            this.timeId = setInterval(() => {
                this.time--;
                this.sendBtnLabel = `${this.time}秒后重发`;
                if (this.time === 0) {
                    clearInterval(this.timeId);
                    this.time = 60;
                    this.sendBtnLabel = '发送验证码';
                }
            }, 1000);
        }
    },
    mounted() {
        this.isShow = true;
    }
}
</script>

<style lang="sass">
.login-wrapper
    position: fixed
    top: 0
    right: 0
    bottom: 0
    left: 0
    background: url(~@/assets/img/login-bg.png) no-repeat center
    background-size: cover
    .login
        display: flex
        position: fixed
        top: 50%
        left: 50%
        width: 800px
        height: 400px
        margin: -200px 0 0 -400px
        background: #fff
        border-radius: 10px
        overflow: hidden
        .l
            width: 480px
            background: url(~@/assets/img/login-left.png) no-repeat center
            background-size: cover
            h1
                margin: 280px 0 0 40px
                font-weight: normal
                font-size: 28px
                color: #fff
        .r
            width: 220px
            padding: 90px 50px 0
            ul
                height: 50px
            li
                float: left
                margin-left: 36px
                font-size: 14px
                color: #666
                cursor: pointer
                transition: color .3s
                &.active
                    position: relative
                    color: #1989FA
                    &:after
                        content: ""
                        position: absolute
                        bottom: -10px
                        left: 50%
                        width: 26px
                        height: 2px
                        margin-left: -13px
                        background: #1989FA
            .input
                position: relative
                margin-bottom: 15px
                .el-input
                    height: 100%
                .el-button
                    position: absolute
                    top: 0
                    right: 0
                    width: 100px
                    height: 100%
                    padding: 12px 10px
                    border-radius: 0 4px 4px 0
            .form > button
                width: 100%

.login-enter, .login-leave-to
    transform: translateY(10%) scale(.8)
    opacity: 0

.login-enter-active, .login-leave-active
    transition: all 1s .5s

</style>
