<template>
    <div class="app">
        <div class="auth-page">
            <div class="auth-title">欢迎回来</div>
            <div class="auth-card">
                <div class="auth-panel auth-panel-email">
                    <div class="panel-header">
                        <div class="panel-title">邮箱登录</div>
                    </div>
                    <div class="auth-input-area">
                        <AppleStyleInput id="email" labelText="请输入邮箱地址" inputType="email" :required="true" v-model="email" />
                        <div class="verification-row">
                            <div class="verification-input">
                                <AppleStyleInput id="captcha" labelText="请输入验证码" inputType="text" :required="true" v-model="captcha" />
                            </div>
                            <button
                                class="send-code-button"
                                @click="sendCaptcha"
                                :disabled="isSending || sendCountdown > 0"
                            >
                                <span v-if="isSending">发送中...</span>
                                <span v-else-if="sendCountdown > 0">{{ sendCountdown }}s</span>
                                <span v-else>发送验证码</span>
                            </button>
                        </div>
                        <div class="terms-text">
                            注册登录即代表已阅读并同意我们的
                            <router-link to="/service-agreement">用户协议</router-link>
                            与
                            <router-link to="/privacy-policy">隐私政策</router-link>
                            ，未注册的邮箱将自动注册
                        </div>
                        <button class="continue-button" @click="loginWithCaptcha" :disabled="isLoading">
                            <span v-if="!isLoading">登录</span>
                            <span v-else>登录中...</span>
                        </button>
                    </div>
                </div>
                <div class="auth-divider"></div>
                <div class="auth-panel auth-panel-wechat">
                    <div class="panel-header">
                        <div class="panel-title">微信登录</div>
                    </div>
                    <div class="wechat-qrcode-wrapper">
                        <div id="wx-qrcode" class="wechat-qrcode-container"></div>
                    </div>
                </div>
            </div>
            <div class="login-with-area">
                <!--<div class="login-with-button" @click="signInWithGoogle">
                    <img class="login-with-button-icon" src="https://cdn-teams-slug.flaticon.com/google.jpg"
                        alt="google" />
                    <div class="login-with-button-text">继续使用 Google 登录</div>
                </div>-->
                <!-- <div class="login-with-button" @click="signInWithWeChat">
                    <img class="login-with-button-icon" src="https://img.icons8.com/color/48/000000/weixing.png" alt="wechat" />
                    <div class="login-with-button-text">继续使用 微信 登录</div>
                </div>-->
                <!-- <div class="login-with-button">
                    <img class="login-with-button-icon"
                        src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSDMKR0m0zmgdmCsLPxh0TKXwhAY_inxpNQHA&s"
                        alt="apple" />
                    <div class="login-with-button-text">继续使用 Apple 登录</div>
                </div> -->
            </div>

        </div>
    </div>
</template>

<script>
import { useToast } from 'vue-toastification'
import AppleStyleInput from '@/components/basic_ui/AppleStyleInput.vue';
import apiClient from '@/api/axios';
import { GoogleAuthProvider, signInWithPopup } from 'firebase/auth'
import { auth } from '@/firebase'
import authService from '@/utils/auth'

export default {
    components: {
        AppleStyleInput
    },
    setup() {
        const toast = useToast()

        return { toast }
    },
    mounted() {
        this.renderQr()
    },
    beforeUnmount() {
        this.clearSendTimer()
    },
    data() {
        return {
            email: '',
            isLoading: false,
            isSending: false,
            captcha: '',
            sendCountdown: 0,
            sendTimer: null,
        }
    },
    methods: {
        async sendCaptcha() {
            if (!this.validateEmail(this.email)) {
                this.toast.error('请输入有效的邮箱地址');
                return;
            }

            if (this.isSending || this.sendCountdown > 0) return;

            this.isSending = true;

            try {
                const response = await apiClient.post('/auth/captcha/send', {
                    email: this.email
                });

                if (response.data.code === 200) {
                    this.toast.success('验证码已发送，请查收邮箱');
                    this.startCountdown();
                } else {
                    this.toast.error(`发送失败: ${response.data.message}`);
                }
            } catch (error) {
                if (error.response) {
                    this.toast.error(`请求错误: ${error.response.data.message}`);
                } else {
                    this.toast.error('网络错误，请检查连接');
                }
            } finally {
                this.isSending = false;
            }
        },
        startCountdown() {
            this.sendCountdown = 60;
            this.clearSendTimer();
            this.sendTimer = setInterval(() => {
                if (this.sendCountdown <= 1) {
                    this.clearSendTimer();
                    this.sendCountdown = 0;
                } else {
                    this.sendCountdown -= 1;
                }
            }, 1000);
        },
        clearSendTimer() {
            if (this.sendTimer) {
                clearInterval(this.sendTimer);
                this.sendTimer = null;
            }
        },
        async loginWithCaptcha() {
            if (!this.validateEmail(this.email)) {
                this.toast.error('请输入有效的邮箱地址');
                return;
            }

            if (!this.captcha) {
                this.toast.error('请输入验证码');
                return;
            }

            this.isLoading = true;

            try {
                const { success, user, error } = await authService.login(this.email, this.captcha);

                if (success) {
                    this.toast.success(`欢迎回来，${user.contact || '用户'}`);
                    this.$router.push('/');
                } else {
                    this.toast.error(error || '登录失败，请重试');
                }
            } catch (e) {
                this.toast.error(e.message || '登录失败，请稍后重试');
            } finally {
                this.isLoading = false;
            }
        },
        async signInWithGoogle() {
            try {
                const provider = new GoogleAuthProvider();
                provider.setCustomParameters({ prompt: 'select_account' });
                const result = await signInWithPopup(auth, provider);
                const idToken = await result.user.getIdToken();
                const { success, user, error } = await authService.loginWithGoogle(idToken);

                if (success) {
                    this.toast.success(`欢迎回来，${user.contact || '用户'}`);
                    this.$router.push('/');
                } else {
                    this.toast.error(error);
                }
            } catch (e) {
                this.toast.error(e.code || e.message);
            }
        },
        signInWithWeChat() {
            const appId = process.env.VUE_APP_WECHAT_APP_ID;
            console.log(`${window.location.origin}/#/wechat-callback`)
            const redirectUri = encodeURIComponent(`${window.location.origin}/#/wechat-callback`);
            const state = Math.random().toString(36).substring(2);
            window.location.href = `https://open.weixin.qq.com/connect/qrconnect?appid=${appId}&redirect_uri=${redirectUri}&response_type=code&scope=snsapi_login&state=${state}#wechat_redirect`;
        },
        validateEmail(email) {
            const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
            return re.test(email);
        },
        genState(len = 24) {
            const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789'
            let s = ''
            for (let i = 0; i < len; i++) s += chars[Math.floor(Math.random() * chars.length)]
            return s
        },
        renderQr() {
            const box = document.getElementById('wx-qrcode')
            if (!box) return
            box.innerHTML = ''
            const state = this.genState()
            sessionStorage.setItem('wx_oauth_state', state)

            if (window.WxLogin) {
                new window.WxLogin({
                    id: "wx-qrcode",
                    appid: process.env.VUE_APP_WECHAT_APP_ID,
                    scope: "snsapi_login",
                    redirect_uri: encodeURIComponent(`${window.location.origin}/#/wechat-callback`),
                    state: Math.random().toString(36).substring(2),
                    style: "black",
                    href: ""
                });
            }
        },


    }
}

</script>

<style scoped>
.app {
    margin-left: 0;
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    background: linear-gradient(180deg, #f7f9ff 0%, #ffffff 100%);
}

.auth-page {
    position: relative;
    width: 780px;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 60px 0;
    gap: 24px;
}

.auth-title {
    font-size: 30px;
    font-weight: 600;
    color: #1d2330;
}

.auth-card {
    width: 100%;
    display: flex;
    gap: 32px;
    padding: 40px 48px;
    background: #ffffff;
    border-radius: 24px;
    box-shadow: 0 24px 80px rgba(15, 46, 145, 0.08);
}

.auth-panel {
    flex: 1;
    display: flex;
    flex-direction: column;
    justify-content: center;
}

.auth-panel-email {
    max-width: 320px;
}

.auth-panel-wechat {
    align-items: center;
    text-align: center;
}

.panel-header {
    margin-bottom: 24px;
}

.panel-title {
    font-size: 22px;
    font-weight: 600;
    color: #1d2330;
}

.auth-input-area {
    display: flex;
    flex-direction: column;
    gap: 16px;
}

.verification-row {
    display: flex;
    align-items: stretch;
    gap: 12px;
}

.verification-input {
    flex: 1;
}

.verification-input .form-group {
    margin-bottom: 0;
}

.send-code-button {
    min-width: 112px;
    height: 50px;
    border-radius: 12px;
    border: none;
    background: #edf0ff;
    color: #5563c1;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.2s ease;
    box-shadow: 0 6px 16px rgba(85, 99, 193, 0.2);
    display: flex;
    align-items: center;
    justify-content: center;
}

.send-code-button:hover {
    background: #dce2ff;
}

.send-code-button:disabled {
    background: #eef1f8;
    color: #98a2c8;
    cursor: not-allowed;
    box-shadow: none;
}

.terms-text {
    font-size: 12px;
    color: #98a2c8;
    line-height: 1.8;
}

.terms-text a {
    color: var(--color-primary);
    margin: 0 4px;
}

.continue-button {
    width: 100%;
    margin-top: 24px;
    background-color: var(--color-primary);
    color: var(--color-secondary);
    border: none;
    padding: 14px 0;
    border-radius: 12px;
    font-size: 15px;
    font-weight: 600;
    box-shadow: 0 12px 30px rgba(88, 119, 255, 0.25);
}

.continue-button:hover {
    background-color: var(--color-primary-hover);
    transition: background-color 0.3s ease;
}

.continue-button:disabled {
    background-color: var(--color-primary-disabled);
    cursor: not-allowed;
    box-shadow: none;
}

.auth-divider {
    width: 1px;
    background: linear-gradient(180deg, rgba(0, 0, 0, 0), rgba(111, 122, 154, 0.16), rgba(0, 0, 0, 0));
}

.wechat-qrcode-wrapper {
    width: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 16px;
}

.wechat-qrcode-container {
    width: 220px;
    height: 260px;
    border-radius: 16px;
    background: #f6f8ff;
    display: flex;
    align-items: center;
    justify-content: center;
    overflow: hidden;
    padding: 12px;
}

.wechat-qrcode-container ::v-deep(iframe) {
    transform: scale(0.72);
    transform-origin: center;
    width: 300px;
    height: 360px;
    border: none;
}

.wechat-qrcode-container .impowerBox .title{
    display: none;
}

.login-with-area {
    display: flex;
    flex-direction: column;
    width: 100%;
    margin-top: 20px;
}

.login-with-button {
    display: flex;
    justify-content: center;
    align-items: center;
    margin-top: 10px;
    border: 1px solid rgba(0, 0, 0, 0.3);
    padding: 10px;
    font-size: 12px;
    border-radius: 5px;
    cursor: pointer;
}

.login-with-button:hover {
    background-color: rgba(0, 0, 0, 0.03);
    transition: background-color 0.3s ease;
}

.login-with-button-icon {
    width: 20px;
    height: 20px;
}

.login-with-button-text {
    margin-left: 10px;
}

@media (max-width: 900px) {
    .auth-page {
        width: 100%;
        padding: 40px 16px;
    }

    .auth-card {
        flex-direction: column;
        padding: 32px 24px;
    }

    .auth-divider {
        width: 100%;
        height: 1px;
        background: linear-gradient(90deg, rgba(0, 0, 0, 0), rgba(111, 122, 154, 0.16), rgba(0, 0, 0, 0));
    }

    .auth-panel-wechat {
        align-items: stretch;
    }

    .wechat-qrcode-wrapper {
        width: 100%;
    }

    .wechat-qrcode-container {
        align-self: center;
    }

    .verification-row {
        flex-direction: column;
        align-items: stretch;
    }

    .send-code-button {
        width: 100%;
        height: 50px;
    }
}
</style>
