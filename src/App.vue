<template>
  <div class="flex justify-center min-h-screen items-center">
    <div class="container p-4">
      <h1 class="mb-10 text-xl md:text-4xl uppercase flex justify-start gap-2 items-center">
      <a href="https://zencod.ru/articles/" target="_blank" class="hover:opacity-50 transition-opacity" title="zencod.ru"><img src="./assets/logo.png" alt="logo zencod.ru" height="40" width="40"></a>
      Генератор .htpasswd
      </h1>
      <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
        <div class="bg-gray-100 rounded-md p-4">
          <div class="grid grid-cols-1 md:grid-cols-2 gap-2">
            <zen-input
              v-model="htpasswd.name"
              label="Название базовой авторизации"
              placeholder="Base"
            />
            <zen-input
              v-model="htpasswd.path"
              label="Путь к файлу .htpasswd"
              placeholder="~/path"
            />
          </div>

          <div class="mt-8">
            <div class="grid grid-cols-5 md:grid-cols-9 gap-2">
              <span class="text-gray-700 col-span-2 md:col-span-4">Логин</span>
              <span class="text-gray-700 col-span-2 md:col-span-4">Пароль</span>
              <span></span>
              <template v-for="(login, keyLogin) in logins" :key="'login-'+keyLogin">
                <zen-input
                  v-model="login.name"
                  placeholder="Логин"
                  class="col-span-2 md:col-span-4"
                />
                <zen-input
                  v-model="login.pass"
                  type-input="password"
                  placeholder="Пароль"
                  class="col-span-2 md:col-span-4"
                />
                <button
                  v-if="keyLogin == logins.length - 1"
                  class="flex py-2 px-4 mt-1 items-center justify-center rounded-md text-white text-xl bg-blue-600 hover:bg-blue-700"
                  @click="addLogin"
                >
                  <icon-add />
                </button>
                <button
                  v-else
                  class="flex py-2 px-4 mt-1 items-center justify-center rounded-md text-white text-xl bg-yellow-600 hover:bg-yellow-700"
                  @click="removeLogin(keyLogin)"
                >
                  <icon-remove />
                </button>
              </template>
            </div>
            <div class="flex flex-col md:flex-row justify-between mt-10 gap-2">
              <label class="block">
                <select v-model="algoritms.selected" class="block w-full mt-1 rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50">
                  <option disabled>Алгоритм шифрования</option>
                  <option v-for="algoritm in algoritms.list" :key="algoritm.name" v-bind:value="algoritm.name">{{ algoritm.desc }}</option>
                </select>
              </label>
              <button @click="createLogins" class="flex justify-center gap-2 py-2 px-4 rounded-md text-white text-xl bg-green-600 hover:bg-green-700">
                Сгенерировать <icon-create />
              </button>
            </div>
          </div>
        </div>
        <div v-if="listLogins" class="bg-gray-100 rounded-md p-4">
          <div class="">
            <div class="font-semibold">.htaccess</div>
            <pre class="font-extralight text-gray-700 border-2 p-2 mt-1 bg-gray-200"><code>{{ htaccess }}</code></pre>
          </div>
          <div class="mt-8">
            <div class="font-semibold">.htpasswd</div>
            <pre class="font-extralight text-gray-700 border-2 p-2 mt-1 bg-gray-200"><code>{{ listLogins }}</code></pre>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import CryptoJS from 'crypto-js/crypto-js';
  import SHA1 from 'crypto-js/sha1';

  import ZenInput from './components/ZenInput.vue'

  import IconAdd from 'virtual:vite-icons/carbon/add'
  import IconRemove from 'virtual:vite-icons/ic/baseline-remove'
  import IconCreate from 'virtual:vite-icons/eos-icons/subscriptions-created-outlined'

  export default {
    components: { IconAdd, IconRemove, IconCreate, ZenInput, },
    data() {
      return {
        listLogins: "",
        htpasswd: {
          name: "Base",
          path: ""
        },
        algoritms: {
          selected: 'sha1',
          list: [
            // { name: 'ssha1', desc: 'SHA1 salted sum in base64'},
            { name: 'sha1', desc: 'SHA1 sum in base64'},
            { name: 'plain', desc: 'Plain Text / No hashing'},
          ]
        },
        logins: [
          { name: "", pass: "" }
        ]
      }
    },
    methods: {
      addLogin() {
        this.logins.push({ name: "", pass: "" })
      },
      removeLogin(index) {
        this.logins.splice(index, 1)
      },
      algo_plain(name, pass) {
        return name + ':' + pass;
      },
      algo_sha1(name, pass) {
        var hash = SHA1(pass);
        return name + ':{SHA}' + hash.toString(CryptoJS.enc.Base64);
      },
      algo_ssha1(name, pass) {
        var salt = CryptoJS.lib.WordArray.random(32/8);
        var sha1 = CryptoJS.algo.SHA1.create();
        sha1.update(pass);
        sha1.update(salt);
        var hash = sha1.finalize();
        return name + ':{SSHA}' + hash.concat(salt).toString(CryptoJS.enc.Base64);
      },
      createLogins() {
        let algoritm = this.algo_ssha1;

        switch (this.algoritms.selected) {
          case "ssha1":
            algoritm = this.algo_ssha1;
            break;
          case "sha1":
            algoritm = this.algo_sha1;
            break;
          case "plain":
            algoritm = this.algo_plain;
            break;
        }

        this.listLogins = this.logins.filter(login => login.name && login.pass).map(login => algoritm(login.name, login.pass)).join('\n')
      }
    },
    computed: {
      htaccess() {
        return "AuthType Basic\nAuthName \""+this.htpasswd.name+"\"\nAuthUserFile \""+this.htpasswd.path+"/.htpasswd\"\nRequire valid-user"
      },

    }
  }
</script>

<style>

</style>
