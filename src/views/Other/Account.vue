<template lang="pug">
main#l-content

  header.c-pageHeader
    button.c-pageHeader__back(@click="$router.go(-1)")
      <img svg-inline src="@/assets/icon/arrow_back.svg" />
    h1 アカウント設定
    <img class="c-pageHeader__logo" svg-inline src="@/assets/logo-short.svg" />

  article.c-article

    h2 表示名の変更
    p 表示名を変更できます。表示名は、複数人プレイ時に他人に表示されるあなたのプレイヤー名です。
    p 現在の表示名:&nbsp;
      b {{ $store.state.user.displayName }}
    input.u-mt16(type='text', v-model='newName', placeholder='新しい表示名を入力してください。')
    button.c-btn.--secondary.u-mt12(v-if='!newName', disabled)
      <img svg-inline src="@/assets/icon/check.svg" />
      | 変更
    button.c-btn.--secondary.u-mt12(v-else, @click='updateName')
      <img svg-inline src="@/assets/icon/check.svg" />
      | 変更
    p.l-toast.c-toast(v-if='msg.updateName') {{ msg.updateName }}

    h2 アカウント削除
    p 作成したキャラクターの情報など、これまでのデータがすべて消去されます。元に戻せませんのでご注意ください。
    button.c-btn.--caution.u-mt12(@click='deleteAccount')
      | アカウントを削除
      <img svg-inline src="@/assets/icon/delete.svg" />
    p.l-toast.c-toast(v-if='msg.deleteAccount') {{ msg.deleteAccount }}

</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import firebase from "@/firebase";

@Component
export default class Account extends Vue {
  // data
  public newName: string = "";
  public msg: any = {
    updateName: "",
    deleteAccount: ""
  };

  // lifecycle hook
  public beforeCreate() {
    if (!this.$store.state.login) {
      // 未ログインの場合
      this.$router.push("/login");
    }
  }

  // methods
  public updateName() {
    const $this = this;
    const user = firebase.auth().currentUser;
    if (!user) {
      return;
    }
    user
      .updateProfile({
        displayName: $this.newName
      })
      .then(() => {
        $this.msg.updateName = "表示名を" + $this.newName + "に変更しました。";
      })
      .catch(error => {
        $this.msg.updateName = "エラーが発生しました。";
      });
  }
  public deleteAccount() {
    const $this = this;
    const confirm: boolean = window.confirm(
      "アカウントを削除すると元には戻せません。削除しますか？"
    );
    const user = firebase.auth().currentUser;
    if (!user || !confirm) {
      return;
    }
    user
      .delete()
      .then(() => {
        // User deleted.
        $this.$store.commit("logout");
        firebase
          .firestore()
          .collection("users")
          .doc(user.uid)
          .delete()
          .catch(error => {
            $this.msg.deleteAccount = "エラーが発生しました。";
            throw error;
          });
      })
      .catch(error => {
        window.console.error(error);
        $this.msg.deleteAccount = "エラーが発生しました。";
      });
  }
}
</script>
