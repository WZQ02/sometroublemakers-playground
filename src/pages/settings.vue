<script setup>
    import { onMounted, ref, getCurrentInstance, watch } from 'vue'
    import { useI18n } from 'vue-i18n'
    const { locale } = useI18n()
    import { stp_store } from '../store.js'

    const ipt1 = ref(null)
    const ipt2 = ref(null)
    const ipt3 = ref(null)
    const ipt4 = ref(null)
    const ipt5 = ref(null)
    const ipt6 = ref(null)
    const ipt7 = ref(null)
    const ipt8 = ref(null)
    const ipt9 = ref(null)
    const ipt10 = ref(null)
    //const ipt11 = ref(null)
    const ipt12 = ref(null)
    const ipt13 = ref(null)
    const ipt14 = ref(null)
    const advanced_settings = ref(null)
    const custom_hls_url = ref("")
    const custom_ws_url = ref("")
    const custom_hls_url_type = ref(0)

    const gCI = getCurrentInstance()
    const adv_options_editable = ref(null);
    const adv_settings_isdisplay = ref(null);
    adv_options_editable.value = 0;
    adv_settings_isdisplay.value = 1;

    function init() {
        switch (stp_store.settings.stp_lang.value) {
            case 1:
                ipt2.value.checked = 1;
                break
            case 2:
                ipt3.value.checked = 1;
                break
            default:
                ipt1.value.checked = 1;
                //break//既不是1也不是2就勾选ipt1
        }
        switch (stp_store.settings.stp_live_lin.value) {
            case 1:
                ipt5.value.checked = 1;
                break
            case 2:
                ipt6.value.checked = 1;
                break
            default:
                ipt4.value.checked = 1;
                //break
        }
        ipt7.value.checked = stp_store.settings.stp_allow_pip.value;
        ipt8.value.checked = adv_options_editable.value = stp_store.settings.adv_set_enabled.value;
        ipt9.value.checked = custom_hls_url_type.value = stp_store.adv_settings.enable_mpegts_player.value;
        ipt10.value.checked = stp_store.adv_settings.disallow_auto_reload_video.value;
        //ipt11.value.checked = stp_store.adv_settings.allow_html_in_chat_content.value;
        ipt12.value.checked = stp_store.settings.danmaku_disabled.value;
        ipt13.value.checked = stp_store.adv_settings.player_native_controls.value;
        ipt14.value.checked = stp_store.settings.browser_fullscreen.value;
        custom_hls_url.value = stp_store.adv_settings.custom_hls_url.value;
        custom_ws_url.value = stp_store.adv_settings.custom_ws_url.value;
    }
    function ipt_select(id) {
        if (id <= 3) {
            stp_store.settings.stp_lang.toggle(id-1);
        } else if (id <= 6) {
            stp_store.settings.stp_live_lin.toggle(id-4);
        }
        switch (id) {
            case 1:
                locale.value = (navigator.language || 'zh').toLocaleLowerCase().split('-')[0]
                break
            case 2:
                locale.value = 'zh'
                break
            case 3:
                locale.value = 'en'
                break
            case 7:
                gCI.proxy?.$bus.emit('change_pip_setting',ipt7.value.checked)//告知video组件画中画设置发生变更
                stp_store.settings.stp_allow_pip.toggle();
                break
            case 8:
                gCI.proxy?.$bus.emit('toggle_native_controls',(ipt8.value.checked && ipt13.value.checked))
                stp_store.settings.adv_set_enabled.toggle();
                adv_options_editable.value = ipt8.value.checked?1:0
                break
            case 9:
                stp_store.adv_settings.enable_mpegts_player.toggle();
                custom_hls_url_type.value = ipt9.value.checked?1:0
                break
            case 10:
                stp_store.adv_settings.disallow_auto_reload_video.toggle();
                break
            case 11:
                stp_store.adv_settings.allow_html_in_chat_content.toggle();
                break
            case 12:
                stp_store.settings.danmaku_disabled.toggle();
                break
            case 13:
                gCI.proxy?.$bus.emit('toggle_native_controls',ipt13.value.checked)//告知video组件是否启用原生控制
                stp_store.adv_settings.player_native_controls.toggle();
                break
            case 14:
                stp_store.settings.browser_fullscreen.toggle();
                break
        }
    }

    function display_adv_settings() {
        if (adv_settings_isdisplay.value) {
            advanced_settings.value.style.display = "none";
            adv_settings_isdisplay.value = 0;
        } else {
            advanced_settings.value.style.display = "";
            adv_settings_isdisplay.value = 1;
        }
    }

    onMounted(() => {
        init();
        if (adv_options_editable.value == 0) {
            display_adv_settings();
        }
        watch(() => custom_hls_url.value,() => {
            stp_store.adv_settings.custom_hls_url.change(custom_hls_url.value)
        })
        watch(() => custom_ws_url.value,() => {
            stp_store.adv_settings.custom_ws_url.change(custom_ws_url.value)
        })
    })
</script>

<template>
    <TransitionGroup name="app_trans"><div id="container" key="home_container">
        <h1>{{$t("settings.title")}}</h1>
        <div id="settings_options">
            <p>
                {{$t("settings.message.1")}}
                <input type="radio" ref="ipt1" name="lang_setting" value="1" @click="ipt_select(1)">{{$t("settings.message.2")}}
                <input type="radio" ref="ipt2" name="lang_setting" value="2" @click="ipt_select(2)">简体中文&nbsp;
                <input type="radio" ref="ipt3" value="3" name="lang_setting" @click="ipt_select(3)">English
            </p>
            <p>
                {{$t("settings.message.3")}}
                <input type="radio" ref="ipt4" name="live_lin" value="1" @click="ipt_select(4)">{{$t("settings.message.4")}}
                <input type="radio" ref="ipt5" name="live_lin" value="2" @click="ipt_select(5)">{{$t("settings.message.5")}}&nbsp;
                <input type="radio" ref="ipt6" name="live_lin" value="3" @click="ipt_select(6)">{{$t("settings.message.6")}}
            </p>
            <p>
                <input type="checkbox" ref="ipt7" class="settings_checkbox" @click="ipt_select(7)">
                &nbsp;{{$t("settings.message.9")}}
            </p>
            <p>
                <input type="checkbox" ref="ipt14" class="settings_checkbox" @click="ipt_select(14)">
                &nbsp;{{$t("settings.message.19")}}
            </p>
            <p>
                <input type="checkbox" ref="ipt12" class="settings_checkbox" @click="ipt_select(12)">
                &nbsp;{{$t("settings.message.17")}}
            </p>
            
            <br>
            <a class="c" @click="display_adv_settings()" v-show="!adv_settings_isdisplay">{{$t("settings.button.1")}}</a>
            <a class="c" @click="display_adv_settings()" v-show="adv_settings_isdisplay">{{$t("settings.button.2")}}</a>
        </div><br>
        <div id="advanced_settings" ref="advanced_settings">
            <p>{{$t("settings.message.10")}}</p>
            <p>
                <input type="checkbox" ref="ipt8" class="settings_checkbox" @click="ipt_select(8)">
                &nbsp;{{$t("settings.message.11")}}
            </p>
            <p v-bind:class="{uneditable:!adv_options_editable}">
                <input type="checkbox" ref="ipt10" class="settings_checkbox" @click="ipt_select(10)" v-bind:disabled="!adv_options_editable">
                &nbsp;{{$t("settings.message.15")}}
            </p>
            <p v-bind:class="{uneditable:!adv_options_editable}">
                <input type="checkbox" ref="ipt9" class="settings_checkbox" @click="ipt_select(9)" v-bind:disabled="!adv_options_editable">
                &nbsp;{{$t("settings.message.14")}}
            </p>
            <p v-bind:class="{uneditable:!adv_options_editable}">
                <input type="checkbox" ref="ipt13" class="settings_checkbox" @click="ipt_select(13)" v-bind:disabled="!adv_options_editable">
                &nbsp;{{$t("settings.message.18")}}
            </p>
            <!--<p v-bind:class="{uneditable:!adv_options_editable}">
                <input type="checkbox" ref="ipt11" class="settings_checkbox" @click="ipt_select(11)" v-bind:disabled="!adv_options_editable">
                &nbsp;{{$t("settings.message.16")}}
            </p>-->
            <!--已改为始终清理聊天室内容，不显示允许html标签选项-->
            <p v-bind:class="{uneditable:!adv_options_editable}">
                {{$t("settings.message.12")}}
                <p>
                    <input type="text" v-show="!custom_hls_url_type" v-bind:placeholder="$t('settings.input.1')" v-model="custom_hls_url" v-bind:disabled="!adv_options_editable">
                    <input type="text" v-show="custom_hls_url_type" v-bind:placeholder="$t('settings.input.3')" v-model="custom_hls_url" v-bind:disabled="!adv_options_editable">
                </p>
            </p>
            <p v-bind:class="{uneditable:!adv_options_editable}">
                {{$t("settings.message.13")}}
                <p></p><input type="text" v-bind:placeholder="$t('settings.input.2')" v-model="custom_ws_url" v-bind:disabled="!adv_options_editable"></p>
        </div>
    </div></TransitionGroup>
</template>

<style scoped>
#settings_options,#advanced_settings {
    animation: appear .6s ease 1;
}
#advanced_settings p {
    transition: 0.25s;
}
#advanced_settings p.uneditable {
    opacity: 0.25;
}
</style>