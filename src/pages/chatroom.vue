<script setup>
    import '../assets/styles/chatroom.css'
    import { onMounted,ref,getCurrentInstance,onActivated, onDeactivated } from 'vue'
    import SvgIcon from '@jamescoyle/vue-icon'
    import { mdiInformation,mdiImage } from '@mdi/js'
    import { stp_store } from '../store.js'

    const usrmsg = ref(null)
    const sendmsg = ref(null)
    const content = ref(null)
    const chgUsrName = ref(null)
    const prompb = ref(null)
    const usrName = ref(null)
    const askforusername_pmpt = ref(null)
    const info = ref(null)
    const currentusername = ref(null)
    const notice = ref(null)
    const quoteselector = ref(null)
    const chatroom_container = ref(null)
    const isshowuserinout = ref(null)
    const ismarkdown = ref(null)
    const roominfo = ref(null)
    const image_upload = ref(null)
    const gCI = getCurrentInstance()

    const image_upload_allowed = ref(0)

    let sendusrmsg = () => {
        if (usrmsg.value.value.length <= 0) {
            gCI.proxy?.$bus.emit('trigger_popup',dystr(str1c))
	    } else if (usrmsg.value.value == "~!@#$%^&*()_+") {
            gCI.proxy?.$bus.emit('chatroom_clean_history');
            usrmsg.value.value = "";
            content.value.innerHTML = "";
        } else if (usrmsg.value.value.length > 256000) {
            gCI.proxy?.$bus.emit('trigger_popup',gCI.proxy?.$t("toasts.3.3"))//msg长度不应超过250k
        } else {
            gCI.proxy?.$bus.emit('chatroom_send',usrmsg.value.value)
		    usrmsg.value.value = "";
	    }
    }
    let askforusername = () => {
	    //更改用户名并刷新本页后，再次修改用户名时把原先用户名自动填写在usrname区
        usrName.value.value = stp_store.chatroom.username.value;
	    prompb.value.style.display = "block";
	    askforusername_pmpt.value.style.display = "block";
    }
    let chgusername = () => {
        let usrNameValue = usrName.value.value;
	    if (usrNameValue == '') {
            gCI.proxy?.$bus.emit('trigger_popup',dystr(str2c))
	    } else if (usrNameValue.length > 50) {//username长度不应超过50
            gCI.proxy?.$bus.emit('trigger_popup',gCI.proxy?.$t("toasts.3.1"))
        } else {
            stp_store.chatroom.username.change(usrNameValue);
            let RealusrName = stp_store.chatroom.username.value;
            gCI.proxy?.$bus.emit('chatroom_chgusrname',RealusrName)
		    noticeUser();
		    askforusername_pmpt.value.style.display = "";
	    }
    }
    //存储“是否显示用户进入、退出”的信息到本地存储
    let isshowuserinout_store = () => {
        stp_store.chatroom.dont_show_userinout.toggle();
        gCI.proxy?.$bus.emit('chatroomisshowuserinout_onchange',isshowuserinout.value.checked)
    }
    let ismarkdown_store = () => {
        stp_store.chatroom.markdown_disabled.toggle();
        gCI.proxy?.$bus.emit('chatroomismarkdown_onchange',ismarkdown.value.checked)
    }
    //显示注意事项
    let noticeUser = () => {
	    notice.value.style.display = "block";
    }
    //关闭注意事项
    let closeprompt = () => {
	    if (notice.value.style.display == "block") {
	    	notice.value.style.animation = "zoomout2 0.2s cubic-bezier(0.6, 0, 1, 0.4) 1";
	    	setTimeout(function(){notice.value.style.display = "none";notice.value.style.animation = ""},192);
	    }
	    if (info.value.style.display == "block") {
	    	info.value.style.animation = "zoomout2 0.2s cubic-bezier(0.6, 0, 1, 0.4) 1";
	    	setTimeout(function(){info.value.style.display = "none";info.value.style.animation = ""},192);
	    }
	    prompb.value.style.animation = "disappear 0.2s ease 1";
	    setTimeout(function(){prompb.value.style.display = "";prompb.value.style.animation = ""},192);
    }
    //显示信息
    let displayinfo = () => {
        let RealusrName = stp_store.chatroom.username.value;
	    prompb.value.style.display = "block";
	    info.value.style.display = "block";
	    currentusername.value.innerText = RealusrName
    }
    //信息页面修改用户名
    let alterusrname = () => {
	    info.value.style.display = "none";
	    askforusername();
    }
    //语录选择器
    let quotechange = () => {
        let selector = quoteselector.value
	    usrmsg.value.value = usrmsg.value.value + selector.options[selector.selectedIndex].value
    }
    //将滚动条位置置于底部
    let goBottom = () => {
        content.value.scrollTop = content.value.scrollHeight;
    }

    //生成随机数改变chatroom页面的部分字符串
    const str1c = [gCI.proxy?.$t("toasts.1.1"), gCI.proxy?.$t("toasts.1.2"), gCI.proxy?.$t("toasts.1.3")];
    const str2c = [gCI.proxy?.$t("toasts.1.4"), gCI.proxy?.$t("toasts.1.5"), gCI.proxy?.$t("toasts.1.6")];
    let dystr = (strxc) => {return strxc[Math.floor(Math.random()*3)];}

    //根据接收的信息更改roominfo（在线人数）数值
    gCI.proxy?.$bus.on('chatroomchgroominfo',(e)=>{
        roominfo.value.innerText = e
    })

    //呈现接受的消息
    gCI.proxy?.$bus.on('chatroomdisplaymsg',(e)=>{
        content.value.appendChild(e);
        goBottom()
    })

    onMounted(() => {
        gCI.proxy?.$bus.emit('req_chatserverbknd')
        gCI.proxy?.$bus.on('chatserverconnected',()=>{
            let RealusrName = stp_store.chatroom.username.value;
	        if (RealusrName == null) {
		        askforusername();
	        } else {
	            //ws.send(`setUsrName=${RealusrName}`);
                //gCI.proxy?.$bus.emit('chatroom_send',`setUsrName=${RealusrName}`)
                //gCI.proxy?.$bus.emit('chatroom_chgusrname',RealusrName)
                //已由bknd更新用户名信息，这里什么都不用做
	        }
	        goBottom();
        })
        //回车发送
        usrmsg.value.addEventListener('keydown',(e) => {
	        if (e.keyCode == 13) {
		        sendusrmsg();
	        }
        });
        //回车更改用户名
        usrName.value.addEventListener('keydown',(e) => {
		    if (e.keyCode == 13) {
			    chgusername();
		    }
	    });
        //读取本地存储并设置存储“是否显示用户进入、退出”的信息
        if (stp_store.chatroom.dont_show_userinout.value) {
            gCI.proxy?.$bus.emit('chatroomisshowuserinout_onchange',1)
            isshowuserinout.value.checked = 1;
        }
        if (stp_store.chatroom.markdown_disabled.value) {
            gCI.proxy?.$bus.emit('chatroomismarkdown_onchange',1)
            ismarkdown.value.checked = 1;
        }

        if (stp_store.chatroom.allow_pic_upload.value) {
            image_upload_allowed.value = 1;
        }
        //监听图片上传事件
        image_upload.value.addEventListener('change',(e)=>{
            if (!e.target.files[0]) {
                return
            }
            const file = e.target.files[0]
            if (file.type.indexOf('image')!=0) {
                gCI.proxy?.$bus.emit('trigger_popup',gCI.proxy?.$t("toasts.3.4"))//不是图片文件
                return
            }
            const reader = new FileReader();
            reader.onload = (ev)=>{
                const b64str = ev.target.result;
                if (b64str.length > 255995) {
                    gCI.proxy?.$bus.emit('trigger_popup',gCI.proxy?.$t("toasts.3.5"))//图片尺寸过大
                } else {
                    gCI.proxy?.$bus.emit('chatroom_send',`![](${b64str})`)
                }
            }
            reader.readAsDataURL(file)
            e.target.value = ''
        })
    })
    onDeactivated(() => {
        gCI.proxy?.$bus.emit('chatroomdeactivated')
    })
    onActivated(() => {
        gCI.proxy?.$bus.emit('chatroomactivated');
        goBottom()
    })
</script>

<template>
    <TransitionGroup name="app_trans"><div id="chatroom_container" ref="chatroom_container" key="chatroom_container">
        <div id="chatcontent" ref="content" name="chatcontent"></div>
        <input type="text" v-bind:placeholder="$t('chatroom.input.1')" id="usrmsg" ref="usrmsg" maxlength="256000"><!--考虑到发送长文本和base64的需求，文本框限制250KB-->
        <div id="panel1">
            <select id="quoteselector" @change="quotechange();" ref="quoteselector">
                <option value="(=・ω・=)">(=・ω・=)</option>
                <option value="(ﾟДﾟ≡ﾟдﾟ)!?">(ﾟДﾟ≡ﾟдﾟ)!?</option>
                <option value="(￣3￣)✧">(￣3￣)✧</option>
                <option value="(≖ ◡ ≖✿)">(≖ ◡ ≖✿)</option>
                <option value="_(≧∇≦」∠)_">_(≧∇≦」∠)_</option>
                <option value="━━━∑(ﾟ□ﾟ*川━">━━━∑(ﾟ□ﾟ*川━</option>
                <option value="(╯°口°)╯(┴—┴">(╯°口°)╯(┴—┴</option>
                <option value="(-_-#)">(-_-#)</option>
                <option value="(๑>؂<๑）">(๑>؂&lt;๑）</option>
                <option value="草">草</option>
                <option value="呵呵">呵呵</option>
                <option value="什么意思？">什么意思？</option>
            </select>
            <button id="sendmsg" ref="sendmsg" @click="sendusrmsg();">{{ $t("chatroom.button.send") }}</button>
            <label v-bind:title="$t('item_title.chatroom.2')">
                <svg-icon type="mdi" :path=mdiInformation @click="displayinfo();" height="24" width="24"></svg-icon>
            </label>
            <label for="image_upload" v-bind:title="$t('item_title.chatroom.1')" v-show="image_upload_allowed">
                <svg-icon type="mdi" :path=mdiImage height="24" width="24"></svg-icon>
                <input type="file" ref="image_upload" id="image_upload" style="display: none;">
            </label>
        </div>
        <div id="prompb" ref="prompb"></div>
        <div id="askforusername" ref="askforusername_pmpt" class="prompt">
            <input type="text" v-bind:placeholder="$t('chatroom.input.2')" id="usrName" ref="usrName" maxlength="50">
            <button id="promptbtn" @click="chgusername();" ref="chgUsrName">{{ $t("chatroom.button.confirm") }}</button>
        </div>
        <div id="notice" ref="notice" class="prompt">
            <span>
                {{ $t("chatroom.message.1") }}<p>{{ $t("chatroom.message.2") }}</p><p>
                {{ $t("chatroom.message.3") }}</p><p>{{ $t("chatroom.message.4") }}</p><p>
                {{ $t("chatroom.message.5") }}</p>
            </span>
            <button id="promptbtn" @click="closeprompt();">{{ $t("chatroom.button.ok_got_it") }}</button>
        </div>
        <div id="info" ref="info" class="prompt">
            {{ $t("chatroom.message.6") }}<span id="currentusername" ref="currentusername" style="font-weight: bold;"></span>&nbsp;&nbsp;<button id="promptbtn" @click="alterusrname();">{{ $t("chatroom.button.change") }}</button><br><br>
            {{ $t("chatroom.message.7") }}<span id="roominfo" ref="roominfo" style="font-weight: bold;"></span><br><br>
            <input type="checkbox" class="chat_infochkbx" name="isshowuserinout" ref="isshowuserinout" @click="isshowuserinout_store()"><span>{{ $t("chatroom.message.8") }}</span><br><br>
            <input type="checkbox" class="chat_infochkbx" name="ismarkdown" ref="ismarkdown" @click="ismarkdown_store()"><span>{{ $t("chatroom.message.9") }}</span><br><br>
            <button id="promptbtn" @click="closeprompt();">{{ $t("chatroom.button.got_it") }}</button>
        </div>
    </div></TransitionGroup>
</template>

<style>
</style>