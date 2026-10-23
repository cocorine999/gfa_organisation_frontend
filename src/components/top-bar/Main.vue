<template>
  <!-- BEGIN: Top Bar -->
  <div class="top-bar">
    <!-- toast notification -->
    <Notification refKey="successNotification" :options="{ duration: 3000 }" class="flex">
      <CheckCircleIcon v-if="message.type === 'success'" class="text-success" />
      <div class="ml-4 mr-4">
        <div :class="{ 'text-red-500 capitalize ': message.type != 'success' }" class="font-medium">{{ message.type }}</div>
        <div class="mt-1 text-slate-500">
          {{ message.message }}
        </div>
      </div>
    </Notification>
    <!-- realtime notifications -->
    <RealtimeNotification />
    <!-- toast notification -->
    <!-- BEGIN: Breadcrumb -->

    <!-- <nav aria-label="breadcrumb" class="hidden mr-auto -intro-x sm:flex">
      <ol class="breadcrumb">
        <li class="breadcrumb-item"><a href="#">Application</a></li>
        <li class="breadcrumb-item active" aria-current="page">Dashboard</li>
      </ol>
    </nav> -->

    <Breadcrumb />

    <!-- BEGIN: Account Menu -->

    <button
      type="button"
      class="relative flex items-center justify-center w-8 h-8 mr-3 rounded-full intro-x hover:bg-white/10"
      title="Notifications non lues"
      @click="toggleNotifications"
    >
      <svg xmlns="http://www.w3.org/2000/svg" class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
        <path stroke-linecap="round" stroke-linejoin="round" d="M15 17h5l-1.405-1.405A2.032 2.032 0 0118 14.158V11a6.002 6.002 0 00-4-5.659V5a2 2 0 10-4 0v.341C7.67 6.165 6 8.388 6 11v3.159c0 .538-.214 1.055-.595 1.436L4 17h5m6 0v1a3 3 0 11-6 0v-1m6 0H9" />
      </svg>
      <span
        v-if="unreadCount > 0"
        class="absolute -top-1 -right-1 min-w-[18px] h-[18px] px-1 flex items-center justify-center text-[10px] font-bold text-white bg-red-500 rounded-full"
      >
        {{ unreadCount > 99 ? "99+" : unreadCount }}
      </span>
    </button>

    <div v-if="showNotifications" class="fixed inset-0 z-40" @click="showNotifications = false"></div>
    <div
      v-if="showNotifications"
      class="absolute right-4 top-[67px] z-50 w-80 max-w-[calc(100vw-2rem)] max-h-96 overflow-y-auto bg-white rounded-md shadow-2xl text-slate-700"
    >
      <div class="flex items-center justify-between px-4 py-3 border-b border-slate-200">
        <span class="font-semibold">Notifications</span>
        <button v-if="notifications.length" type="button" class="text-xs font-semibold text-red-500" @click="clearNotifications">
          Effacer tout
        </button>
      </div>
      <div v-if="!notifications.length" class="px-4 py-6 text-sm text-center text-slate-500">
        Aucune notification
      </div>
      <button
        v-for="notification in notifications"
        :key="notification.id"
        type="button"
        class="w-full px-4 py-3 text-left border-b border-slate-100 hover:bg-slate-50"
        @click="readNotification(notification)"
      >
        <div class="text-sm">
          <span class="font-semibold uppercase text-indigo-700">{{ notification.module }} :</span>
          {{ notification.texte }}
        </div>
        <div class="mt-1 text-xs text-slate-400">{{ notification.created_at }}</div>
      </button>
    </div>

    <Dropdown class="w-8 h-8 intro-x">
      <DropdownToggle tag="div" role="button" class="dropdown-toggle image-fit zoom-in">
        <div class="flex items-center justify-center w-8 h-8 bg-blue-400 rounded-full shadow-lg cursor-pointer">
          <img v-if="usersProfileImage" :src="usersProfileImage" alt="" />
          <span v-else class="p-2 text-sm font-bold text-white uppercase"> {{ currentUsers.initiale }} </span>
        </div>
      </DropdownToggle>
      <DropdownMenu class="w-56">
        <DropdownContent class="text-white bg-primary">
          <DropdownHeader tag="div" class="!font-normal">
            <div class="font-medium">
              <!-- {{  currentUsers.prenom }} -->
              <span> {{ currentUsers.nom }} </span>
            </div>
            <div class="text-xs text-white/70 mt-0.5 dark:text-slate-500">
              {{ currentUsers.role }}
            </div>
          </DropdownHeader>
          <DropdownDivider class="border-white/[0.08]" />
          <DropdownItem class="dropdown-item hover:bg-white/5">
            <span @click="gotoProfil" class="flex w-full space-y-2" to=""> <UserIcon class="w-4 h-4 mr-2" /> Profil </span>
          </DropdownItem>
          <DropdownItem class="dropdown-item hover:bg-white/5">
            <router-link class="flex w-full space-y-2" to="/dashboard/resetPassword"> <LockIcon class="w-4 h-4 mr-2" /> Réinitialiser le Mot de Passe </router-link>
          </DropdownItem>

          <DropdownDivider class="border-white/[0.08]" />
          <DropdownItem class="dropdown-item hover:bg-white/5">
            <span class="flex items-center space-x-2 cursor-pointer" @click="logout"> <ToggleRightIcon class="w-4 h-4 mr-2" /> Se déconnecter </span>
          </DropdownItem>
        </DropdownContent>
      </DropdownMenu>
    </Dropdown>

    <!-- END: Account Menu -->
  </div>
  <!-- END: Top Bar -->
</template>

<script setup>
import { ref, reactive, onMounted, provide, computed } from "vue";
import { useAuthStore } from "@/stores/modules/authentification";
import AuthentificationService from "@/services/modules/auth.service";
import { useRouter, useRoute } from "vue-router";
import Breadcrumb from "../Breadcrumb.vue";
import RealtimeNotification from "@/components/Notification.vue";
import vuexStore from "@/stores/index";
import NotificationService from "@/services/modules/notification.service.js";
import { API_BASE_URL } from "@/services/configs/environment";
const router = useRouter();
const route = useRoute();
const searchDropdown = ref(false);
const showSearchDropdown = () => {
  searchDropdown.value = true;
};
const hideSearchDropdown = () => {
  searchDropdown.value = false;
};

const store = useAuthStore();
const currentUsers = reactive({});
const successNotification = ref();
const initiale = ref();

const message = reactive({
  type: "success",
  message: "",
});

const usersProfileImage = ref("");
const unreadCount = computed(() => vuexStore.getters.getCount);
const showNotifications = ref(false);
const notifications = ref([]);

const loadUnreadNotifications = () => {
  NotificationService.get()
    .then((response) => {
      const datas = response?.data?.data ?? [];
      notifications.value = Array.isArray(datas) ? datas : [];
      vuexStore.commit("updateCounter", notifications.value.length);
    })
    .catch(() => {});
};

const toggleNotifications = () => {
  showNotifications.value = !showNotifications.value;
  if (showNotifications.value) {
    loadUnreadNotifications();
  }
};

const readNotification = (notification) => {
  NotificationService.checkMessage({ id: notification.id })
    .then(() => {
      notifications.value = notifications.value.filter((item) => item.id !== notification.id);
      vuexStore.commit("updateCounter", notifications.value.length);
    })
    .catch(() => {});
};

const clearNotifications = () => {
  NotificationService.destroy()
    .then(() => {
      notifications.value = [];
      vuexStore.commit("updateCounter", 0);
    })
    .catch(() => {});
};

onMounted(() => {
  loadUnreadNotifications();
  const usersInfo = JSON.parse(localStorage.getItem("authenticateUser"));

  // console.log('usersInfo' , usersInfo)

  if (usersInfo) {
    // usersProfileImage.value = API_BASE_URL + usersInfo.users.profil
    currentUsers.nom = usersInfo.nom;
    // currentUsers.nom = "test";
    // currentUsers.prenom = usersInfo.users.prenom
    currentUsers.role = usersInfo.role[0].nom;
    //  + currentUsers.prenom[0]
    let storeInitial = currentUsers.nom[0] + "";
    storeInitial = storeInitial.toUpperCase();
    initiale.value = storeInitial;
    currentUsers.initiale = initiale.value;
  }
});

const logout = () => {
  localStorage.removeItem("bsdInfo");
  localStorage.removeItem("access_token");
  localStorage.removeItem("authenticateUser");
  message.type = "success";
  message.message = "Vous êtes déconnecté";
  successNotificationToggle();

  setTimeout(() => {
    router.push("/");
  }, 100);

  AuthentificationService.logout()
    .then(() => {
      store.resetInfo();
      message.type = "success";
      message.message = "Vous êtes déconnecté";
      successNotificationToggle();
    })
    .catch((error) => {
      setTimeout(() => {
        router.push("/");
      }, 100);
      console.log(error);

      // Afficher le message d'erreur du serveur
      const errorMessage = error.response?.data?.message || "Erreur de déconnexion";
      toast.error(errorMessage);
    });
};

const gotoProfil = () => {
  router.push("/dashboard/profil");
};
provide("bind[successNotification]", (el) => {
  // Binding
  successNotification.value = el;
});
const successNotificationToggle = () => {
  // Show notification
  successNotification.value.showToast();
};
</script>
