<template>
<div class="top-header">
    <h2>设备管理系统</h2>

    <el-dropdown class="user" @command="onClick">
        <span class="el-dropdown-link">
            {{ $store.state.username }}<i class="el-icon-arrow-down el-icon--right"></i>
        </span>
        <el-dropdown-menu slot="dropdown">
            <el-dropdown-item :command="'changePw'">修改密码</el-dropdown-item>
            <el-dropdown-item command="logout">退出登陆</el-dropdown-item>
        </el-dropdown-menu>
    </el-dropdown>

    <el-dialog
        title="修改密码"
        :visible="showChangePw"
        @close="showChangePw = false"
        :width="'500px'"
    >
        <div class="change-pw-form">
            <el-form label-width="100px">
                <el-form-item label="旧密码">
                    <el-input type="password" v-model="formData.oldPassword"></el-input>
                </el-form-item>
                <el-form-item label="新密码">
                    <el-input type="password" v-model="formData.newPassword1"></el-input>
                </el-form-item>
                <el-form-item label="确认新密码">
                    <el-input type="password" v-model="formData.newPassword2"></el-input>
                </el-form-item>
            </el-form>
        </div>
        <div slot="footer" class="dialog-footer">
            <el-button @click="showChangePw = false">取消</el-button>
            <el-button type="primary" @click="onChangePw">确定</el-button>
        </div>
    </el-dialog>
</div>
</template>

<script>
import axios from 'axios';
export default {
    data() {
        return {
            showChangePw: false,
            formData: {
                oldPassword: '',
                newPassword1: '',
                newPassword2: '',
            }
        }
    },
    methods: {
        logout() {
            axios.post('/equipment/logout').then(() => {
                this.$router.push('/login');
            });
        },
        changePw() {
            this.showChangePw = true;
        },
        onClick(val) {
            // if (val === 'changePw') {
            //     this.changePw();
            // }
            this[val]();

            // obj = { a: function() {} };
            // let val = 'a';
            // obj.a();
            // obj[val]();

            // callbackName = 'fn';
            // window[callbackName] = function() {};
            // function fn() {}
        },
        onChangePw() {
            const { oldPassword, newPassword1, newPassword2 } = this.formData;
            let msg = '';
            if (!oldPassword) {
                msg = '请填写旧密码';
            } else if (!newPassword1 || !newPassword2) {
                msg = '请填写新密码';
            } else if (newPassword1 !== newPassword2) {
                msg = '两次新密码填写必须一致';
            }
            if (msg) return this.$message.warning(msg);
            axios.post('/equipment/password-reset', {
                oldPw: oldPassword,
                newPw: newPassword1
            }).then(() => {
                this.$router.push('/login');
                this.$message.success('密码修改成功，请重新登陆');
            });
        }
    }
};
</script>

<style lang="sass">
.top-header
    position: relative
    height: 60px
    background: #0079C4
    padding: 0 20px
    color: #fff
    h2
        line-height: 60px
        font-size: 18px
        padding-left: 30px
        background: url(~@/assets/img/logo.png) no-repeat left center
        background-size: 18px auto
    .user
        position: absolute
        top: 0
        right: 20px
        .el-dropdown-link
            line-height: 60px
            color: #fff
</style>
