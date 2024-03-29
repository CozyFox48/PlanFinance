<template>
  <div class="flex justify-center items-center h-screen bg-gray-200 px-6">
    <div class="p-8 max-w-sm w-full bg-white shadow-md rounded-md">
      <div class="flex justify-center items-center">
        <go-plan-logo />
      </div>

      <br />
      <br />

      <div v-if="hasClientKey === 'anonymous'" class="flex justify-center">
        <google-button @clicked="signInGoogle" />
      </div>
      <div v-else-if="hasClientKey === 'yes'">
        <unlock-master-key @keyValid="clientKeyValid" />
      </div>
      <div v-else-if="hasClientKey === 'no'">
        <create-master-key @keyValid="clientKeyValid" />
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { AuthStore } from '@/store';
import { User } from '@common/models';
import { ProcessUtils } from '@goplan-finance/utils';
import { defineComponent, getCurrentInstance, inject, ref } from 'vue';
import { useRouter } from 'vue-router';
import GoPlanLogo from '@components/base/GoPlanLogo.vue';
import CreateMasterKey from './CreateMasterKey.vue';
import GoogleButton from './googleButton.vue';
import UnlockMasterKey from './UnlockMasterKey.vue';

export default defineComponent({
  components: {
    GoPlanLogo,
    GoogleButton,
    CreateMasterKey,
    UnlockMasterKey,
  },
  setup() {
    const app = getCurrentInstance();
    const gapi = app.appContext.config.globalProperties.$gapi;
    const router = useRouter();
    const authStore = inject('$authStore') as AuthStore;

    const hasClientKey = ref('anonymous');

    const signInGoogle = async () => {
      try {
        // const client = await gapi.getGapiClient() // @todo this is a hack,
        const auth = await gapi.getAuthInstance();
        await auth.signIn({
          prompt: 'select_account',
          grant_type: 'authorization_code',
          scope: 'profile',
        });

        const currentGoogleUser = auth.currentUser.get();
        await User.logInWith('google', {
          authData: {
            id: currentGoogleUser.getId(),
            id_token: currentGoogleUser.getAuthResponse().id_token,
          },
        });

        // const profile           = currentGoogleUser.getBasicProfile()
        // await Parse.Cloud.run('User--updateGoogleInfo', {
        //   info: {
        //     id        : profile.getId(),
        //     fullName  : profile.getName(),
        //     firstName : profile.getGivenName(),
        //     lastName  : profile.getFamilyName(),
        //     imageUrl  : profile.getImageUrl(),
        //     email     : profile.getEmail(),
        //   },
        // })
        const user = (await User.currentAsync()) as User;

        if (!user) {
          return;
        }

        hasClientKey.value = (await authStore.hasClientKey()) ? 'yes' : 'no';
      } catch (error) {
        console.error(error);
        // this.$notifyError({
        //   title : 'Authentification failed',
        //   text  : error,
        // })
      }
    };

    const clientKeyValid = async () => {
      // @todo Show spinner
      await ProcessUtils.sleep(1000);
      await router.push({ name: 'transactions' });
    };

    return {
      hasClientKey,
      clientKeyValid,
      signInGoogle,
      GoogleButton,
    };
  },
});
</script>
