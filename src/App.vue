<template>
  <v-app dark>
    <v-navigation-drawer permanent clipped app>
      <v-list>
        <template v-for="(item, i) in menu">
          <v-divider dark v-if="item.divider" class="my-4" :key="i" v-show="!notShow[item.name]"></v-divider>
          <v-list-tile :key="i" v-else @click="changeView(item.name)" active-class="grey darken-4" v-model="isCurrent[item.name]" v-show="!notShow[item.name]">
            <v-list-tile-action>
              <v-icon>{{ item.icon }}</v-icon>
            </v-list-tile-action>
            <v-list-tile-content>
              <v-list-tile-title class="grey--text">
                {{ $t('common.menu.' + item.name) }}
              </v-list-tile-title>
            </v-list-tile-content>
          </v-list-tile>
        </template>
      </v-list>
    </v-navigation-drawer>
    <v-toolbar :class="headerClass" app fixed clipped-left>
      <span class="title">
        <i class="qtum-icon qtum-icon-logo"></i>
        <span class="text">QTUM</span>
        <span @click="changeView('settings')">
          --{{ $t('common.' + network) }}
        </span>
        <v-btn flat @click="changeView('settings')" v-if="mode != 'normal'">
          {{ $t('common.mode.' + mode) }}
        </v-btn>
      </span>
    </v-toolbar>
    <main>
      <v-content>
        <v-container fluid fill-height justify-center>
          <v-layout row wrap>
            <v-flex xs10 offset-xs1>
              <create-wallet :view="isCurrent['create']" @created="setWallet" v-show="isCurrent['create']"></create-wallet>
              <create-mnemonic :view="isCurrent['create_from_mnemonic']" @created="setWallet" v-show="isCurrent['create_from_mnemonic']"></create-mnemonic>
              <restore-wallet @restored="setWallet" v-show="isCurrent['restore_from_mnemonic']"></restore-wallet>
              <restore-wif @restored="setWallet" v-show="isCurrent['restore_from_wif']"></restore-wif>
              <restore-mobile @restored="setWallet" v-show="isCurrent['restore_from_mobile']"></restore-mobile>
              <restore-key-file @restored="setWallet" v-show="isCurrent['restore_from_key_file']"></restore-key-file>
              <view-wallet :view="isCurrent['view']" v-if="isCurrent['view']"></view-wallet>
              <view-tx :view="isCurrent['transactions']" v-if="isCurrent['transactions']"></view-tx>
              <safe-send @send="setWallet" v-if="isCurrent['safe_send']"></safe-send>
              <send @send="setWallet" v-if="isCurrent['send']"></send>
              <request-payment v-if="isCurrent['request_payment']"></request-payment>
              <dump-key-file v-if="isCurrent['dump_as_key_file']"></dump-key-file>
              <create-contract v-if="isCurrent['create_contract']"></create-contract>
              <send-to-contract v-if="isCurrent['send_to_contract']"></send-to-contract>
              <call-contract v-if="isCurrent['call_contract']"></call-contract>
              <config v-if="isCurrent['settings']"></config>
            </v-flex>
          </v-layout>
        </v-container>
      </v-content>
    </main>
    <notify></notify>
    <warning></warning>
  </v-app>
</template>

<script>
import Vue from 'vue'

//Components
import Notify from 'components/Notify'
import Warning from 'components/Warning'
import CreateWallet from 'controllers/Create'
import CreateMnemonic from 'controllers/CreateMnemonic'
import RestoreWallet from 'controllers/Restore'
import RestoreWif from 'controllers/RestoreWif'
import RestoreMobile from 'controllers/RestoreMobile'
import RestoreKeyFile from 'controllers/RestoreKeyFile'
import ViewWallet from 'controllers/View'
import ViewTx from 'controllers/ViewTx'
import SafeSend from 'controllers/SafeSend'
import Send from 'controllers/Send'
import RequestPayment from 'controllers/RequestPayment'
import DumpKeyFile from 'controllers/DumpKeyFile'
import CreateContract from 'controllers/CreateContract'
import SendToContract from 'controllers/SendToContract.vue'
import CallContract from 'controllers/CallContract.vue'
import Config from 'controllers/Config'

import config from 'libs/config'
import webWallet from 'libs/web-wallet'
import i18n from 'libs/i18n'

export default {
  name: 'app',
  i18n,
  data() {
    return {
      wallet: false,
      current: 'create',
      network: config.getNetwork(),
      mode: config.getMode(),
      menu: [
        { icon: 'add', name: 'create' },
        { icon: 'assignment', name: 'create_from_mnemonic' },
        { icon: 'sms', name: 'restore_from_mnemonic' },
        { icon: 'create', name: 'restore_from_wif' },
        { icon: 'phonelink_lock', name: 'restore_from_mobile' },
        { icon: 'cloud_upload', name: 'restore_from_key_file' },
        { divider: true, name: 'wallet' },
        { icon: 'account_balance_wallet', name: 'view' },
        { icon: 'list', name: 'transactions' },
        { icon: 'security', name: 'safe_send' },
        { icon: 'repeat', name: 'send' },
        { icon: 'undo', name: 'request_payment' },
        { icon: 'cloud_download', name: 'dump_as_key_file' },
        { divider: true, name: 'contract' },
        { icon: 'gavel', name: 'create_contract' },
        { icon: 'publish', name: 'send_to_contract' },
        { icon: 'play_circle_filled', name: 'call_contract' },
        { divider: true, name: 'disc' },
        { icon: 'settings', name: 'settings' },
      ],
      notifyList: {}
    }
  },
  computed: {
    isCurrent() {
      return { [this.current]: true }
    },
    notShow() {
      return {
        view: this.mode == 'offline' || this.wallet == false,
        transactions: this.mode == 'offline' || this.wallet == false,
        wallet: this.mode == 'offline' && this.wallet == false,
        safe_send: this.mode == 'offline' && this.wallet == false,
        send: this.mode == 'offline' || this.wallet == false,
        request_payment: this.wallet == false,
        dump_as_key_file: this.wallet == false,
        contract: this.mode == 'offline' || this.wallet == false,
        create_contract: this.mode == 'offline' || this.wallet == false,
        send_to_contract: this.mode == 'offline' || this.wallet == false,
        call_contract: this.mode == 'offline' || this.wallet == false,
      }
    },
    headerClass() {
      return this.mode == 'normal' ? 'cyan' : 'orange'
    }
  },
  components: {
    Notify,
    Warning,
    CreateWallet,
    CreateMnemonic,
    RestoreWallet,
    RestoreWif,
    RestoreMobile,
    RestoreKeyFile,
    ViewWallet,
    ViewTx,
    SafeSend,
    Send,
    RequestPayment,
    DumpKeyFile,
    CreateContract,
    SendToContract,
    CallContract,
    Config,
  },
  methods: {
    setWallet() {
      this.wallet = webWallet.getWallet()
      this.wallet.init()
      if (this.wallet) {
        if (this.mode == 'offline') {
          this.current = 'request_payment'
        }
        else {
          this.current = 'view'
        }
      }
    },
    changeView(name) {
      this.current = name
    },
    error(msg) {
      this.addNotify(msg, 'error')
    },
    success(msg) {
      this.addNotify(msg, 'success')
    },
    addNotify(msg, type) {
      let notifyId = [msg, type].join('_')
      let notify = {
        msg: msg.split(' ').reduce((msg, current) => {
          let tmsg = this.$t('common.notify.' + current)
          tmsg = (tmsg == 'common.notify.' + current) ? ' ' + current : tmsg
          return msg + tmsg
        }, ''),
        type
      }
      if (this.notifyList[notifyId]) {
        clearTimeout(this.notifyList[notifyId].timer)
      }
      Vue.set(this.notifyList, notifyId, notify)
      this.notifyList[notifyId].timer = setTimeout(() => {Vue.delete(this.notifyList, notifyId)}, 10000)
    }
  }
}
</script>
