<template>
	<div class="loginByMobile">
		<!--header-->
		<van-nav-bar :title="bind ? '手机绑定' : '手机登录(免注册)'" class="white" left-arrow @click-left="onClickLeft" />
		<!--main-->
		<div class="main" style="position: relative;">
			<van-cell-group style="position: absolute; z-index: 999;width: 100%;" :border="false">
			  <van-field
			    v-model="phone"
			    center
			    clearable
			    label="+86"
			    placeholder="输入手机号"

			  />
			  <van-field
			    v-model="sms"
			    center
			    clearable
			    label="验证码"
			    placeholder="输入验证码"
			  >
			    <van-button slot="button" size="small" @click="getSMS" class="plain">{{count_text}}</van-button>
			  </van-field>
			  <div class="protocol">
			  	<!--<input :checked="checked" @click="checked = !checked" type="checkbox" />-->
			  	<i class="iconfont" :class="[checked ? 'icon-checkoutline02' : 'icon-check02']" @click="checked = !checked"></i>
			  	<Protocol showTip="2"></Protocol>
			  </div>
			  <van-button round  size="large" class="blue" @click="login">登录</van-button>
			</van-cell-group>
		</div>
		<!--footer-->
		<div class="footer" v-show="showFooter">
			<img src="../../assets/img/logo_h.png"/>
			<div class="logo_name">广州柒度信息科技有限公司</div>
		</div>
	</div>
</template>

<script>
	import api from '@/api/api'
	import Protocol from '@/components/protocol'
	import utils from '@/utils/utils'
	export default{
		components:{
			Protocol
		},
		data(){
			return{
				showFooter: true,
				phone:'',
				sms:'',
				time:60,
				count_text:'获取验证码',
				canSMS:true,//判断验证码按钮状态
				isGetSMS:true,//已点击获取验证码按钮
				checked:false,
				bind:this.$route.query.bind,
        user_referee: this.$route.query.user_referee,
			}
		},
    watch: {
      sms(val) {
        if (val.length > 4) {
          this.sms = val.slice(0, 4)
        }
      }
    },
		mounted(){
			window['onSMSReceiveSuccess'] = res => {
        if(res.code){
          this.sms = res.code
        }
			}
		},
		methods:{
			toggleLogo () {
				this.showFooter = false
			},
			onClickLeft(){
				this.$router.push({
					path:'/login'
				})
			},
			getSMS(){
        if(this.canSMS){
					var phoneReg = /^(13[0-9]|14[579]|15[0-3,5-9]|16[6]|17[0135678]|18[0-9]|19[89])\d{8}$/;
            if (!phoneReg.test(this.phone)) {
                this.$toast('手机号码不合法');
            }else{
              this.canSMS = false;
              this.$fetch('user_code_send', {'user_phone':this.phone}).then(rs => {
                this.$toast('验证码已发送');
                this.countDown();
                this.isGetSMS = true;
              }).catch(e=>{
                this.canSMS = true;
              })
            }
				}else{
					this.$toast('请勿频繁操作')
				}
			},
			countDown() {
				let that = this
				that.count_text = that.time + '秒'
				if(that.time === 0) {
					that.count_text = '获取验证码'
					that.time = 60
					that.canSMS = true
				} else {
					that.count_text = (that.time--) + '秒'
					setTimeout(that.countDown, 1000);
				}
			},
			login(){
				var phoneReg = /^(13[0-9]|14[579]|15[0-3,5-9]|16[6]|17[0135678]|18[0-9]|19[89])\d{8}$/;
        if (!phoneReg.test(this.phone)) {
          this.$toast('手机号码不合法');
        }else if(!this.isGetSMS){
          this.$toast('请先获取验证码');
        }else if(this.sms.length !== 4){
          this.$toast('验证码不合法');
        }else if(!this.checked){
          this.$toast('请勾选并同意帮家洁平台相关协议');
        }else{
          let apiurl = 'user_login_msn';
          let data = {};
          data.verify_code = this.sms;
          if(this.bind){
            apiurl = 'user_bind_phone';
            data.is_oauth = 1
            data.user_phone = this.phone;
            data.oauth_account = this.$route.query.oauth_account;
            data.oauth_type = this.$route.query.oauth_type
          }else{
            this.user_referee ? data.user_referee = this.user_referee : 1
            data.mobile = this.phone
          }
          this.$fetch(apiurl, data).then(rs => {
            utils.loginToJPush(rs.user_id)
            this.$store.commit('login', rs.token);
            this.$router.replace({
              path:'/home'
            })
          })
        }
			},


		}
	}
</script>

<style lang="less" scoped>
	@import "../../assets/icon/iconfont.css";
	.loginByMobile{
		position: relative;
		height: 100%;
		.main{
			/*border-top: 1px solid #efefef;*/
			.protocol{
				display: flex;
				align-items: center;
				font-size: .12rem;
				padding: 0 .2rem;
				i{
					color: #1BB9ED;
					margin-right: .1rem;
				}
			}
			.van-button.plain{
				border: 1px solid #18b4ed;
				color: #18b4ed;
				font-size: .14rem;
			}
			.van-button.blue{
				margin: .3rem .2rem 0;
				background: #18b4ed;
				color: #fff;
				font-size: .18rem;
				width: calc(100% - .4rem);
			}
		}
		.footer{
			position: absolute;
			bottom: .15rem;
			text-align: center;
			width: 100%;
			>img{
				width: .89rem;
				height: auto;
			}
		}
		.van-cell:not(:last-child)::after{
			border: 0;
		}
	}
</style>
<style>
	.loginByMobile .van-field__body{
		padding: .1rem 0;
		border-bottom: 1px solid #e1f0ff;
	}
</style>
