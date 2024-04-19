<template>
  <!-- Nav -->
  <nav id="navbar-top" class="navbar navbar-expand-lg bg-body-tertiary px-3 mb-3 jn-navbar-top"
    :class="{ 'dark-mode-nav navbar-dark bg-dark': isDarkMode }">
    <div class="jn-logo">

      <a class="navbar-brand d-flex align-items-center align-content-center" :class="{ 'text-white': isDarkMode }"
        href="#" @click="handleLogoClick">
        <img src="../../public/logo.svg" alt="icon ivg" height="23" width="23">
        <span class=" fw-bold  ">IP</span>
        <span class="fw-lighter"></span>
        <span class="fw-lighter" :class="{
          'background-animation-dark': !loaded && isDarkMode,
          'background-animation-light': !loaded && !isDarkMode
        }">ing</span>
      </a>

      <div class="btn-group mx-1" :data-bs-theme="isDarkMode ? 'dark' : 'light'">
        <button type="button" class="btn btn-sm dropdown-toggle jn-button" data-bs-toggle="dropdown"
          aria-expanded="false" aria-label="Language Selection">
          <i class="bi bi-translate"></i>
        </button>
        <ul class="dropdown-menu">
          <li><a class="dropdown-item" href="?hl=zh" @click="$trackEvent('Nav', 'ToggleClick', 'LanguageChange')"><i
                class="fi fi-cn"></i> 中文</a></li>
          <li><a class="dropdown-item" href="?hl=en" @click="$trackEvent('Nav', 'ToggleClick', 'LanguageChange')"><i
                class="fi fi-us"></i> English</a></li>
          <li><a class="dropdown-item" href="?hl=fr" @click="$trackEvent('Nav', 'ToggleClick', 'LanguageChange')"><i
                class="fi fi-fr"></i> Français</a></li>
        </ul>
      </div>

      <div>
        <input type="checkbox" class="jn-checkbox" aria-label="Toggle Dark Mode" id="toggleBtn" v-model="isDarkMode"
          @click="toggleDarkMode" />
        <label class="switch" for="toggleBtn">
          <i class="bi bi-moon-stars text-light"></i>
          <i class="bi bi-brightness-high text-warning "></i>
          <div class="ball"></div>
        </label>
      </div>

    </div>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNavAltMarkup"
      aria-controls="navbarNavAltMarkup" aria-expanded="false" aria-label="Toggle navigation">
      <span class="navbar-toggler-icon bg-transparent "></span>
    </button>
    <div class="collapse navbar-collapse justify-content-end" id="navbarNavAltMarkup">
      <div class="navbar-nav ">
        <a class="nav-link" :class="{ 'text-white jn-deactive': isDarkMode }" href="#IPInfo"
          @click="collapseNav(); $trackEvent('Nav', 'NavClick', 'IPInfo')"> {{ $t('nav.IPinfo') }}</a>
        <a class="nav-link" :class="{ 'text-white jn-deactive': isDarkMode }" href="#Connectivity"
          @click="collapseNav(); $trackEvent('Nav', 'NavClick', 'Connectivity')"> {{
            $t('nav.Connectivity') }}</a>
        <a class="nav-link" :class="{ 'text-white jn-deactive': isDarkMode }" href="#WebRTC"
          @click="collapseNav(); $trackEvent('Nav', 'NavClick', 'WebRTC')"> {{ $t('nav.WebRTC') }}</a>
        <a class="nav-link" :class="{ 'text-white jn-deactive': isDarkMode }" href="#DNSLeakTest"
          @click="collapseNav(); $trackEvent('Nav', 'NavClick', 'DNSLeakTest')"> {{
            $t('nav.DNSLeakTest') }}</a>
        <a class="nav-link" :class="{ 'text-white jn-deactive': isDarkMode }" href="#SpeedTest"
          @click="collapseNav(); $trackEvent('Nav', 'NavClick', 'SpeedTest')"> {{ $t('nav.SpeedTest')
          }}</a>
        <a class="nav-link" :class="{ 'text-white jn-deactive': isDarkMode }" href="#AdvancedTools"
          @click="collapseNav(); $trackEvent('Nav', 'NavClick', 'AdvancedTools')"> {{ $t('nav.AdvancedTools') }}</a>
      </div>
      <a :href="$t('page.footerLink')" class="btn jn-fs" id="githubStars"
        :class="{ 'btn-outline-light' : isDarkMode, 'btn-dark': !isDarkMode, 'mt-2': isMobile, 'ms-2': !isMobile}" target="_blank"
        @click="$trackEvent('Footer', 'FooterClick', 'Github');" aria-label="Github">
        <div><i class="bi bi-github"></i></div>
        <div class="row flex-column ">
          <TransitionGroup name="slide-fade">
            <span key="default" class="col-12 jn-w" v-if="githubStars === 0">&nbsp;GitHub</span>
            <span key="stars" class="col-12 jn-w" v-if="githubStars > 0">
              &nbsp;{{ githubStars }}
              <i class="bi bi-star-fill" :class="[isDarkMode ? 'redstar' : 'yellowstar']"></i>
            </span>
          </TransitionGroup>

        </div>
      </a>
    </div>
  </nav>
</template>

<script>
import { computed } from 'vue';
import { useStore } from 'vuex';

export default {
  name: 'NavBar',

  // 引入 Store
  setup() {
    const store = useStore();
    const isDarkMode = computed(() => store.state.isDarkMode);
    const isMobile = computed(() => store.state.isMobile);

    return {
      isDarkMode,
      isMobile
    };
  },

  data() {
    return {
      loaded: false,
      githubStars: 0,
    }
  },
  methods: {

    //获取 GitHub stars
    async getGitHubStars() {
      const url = `https://api.github.com/repos/jason5ng32/MyIP`;

      try {
        const response = await fetch(url);
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        const data = await response.json();
        setTimeout(() => {
          this.githubStars = data.stargazers_count;
        }, 1000);
      } catch (error) {
        console.error('Failed to fetch Github data:', error);
        this.githubStars = 0;
      }
    },

    // 切换暗黑模式
    toggleDarkMode() {
      this.$store.commit('toggleDarkMode');
      this.updateBodyClass();
      this.PWAColor();
      this.$trackEvent('Nav', 'ToggleClick', 'DarkMode');
    },

    // 收起导航栏
    collapseNav() {
      document.querySelector('#navbarNavAltMarkup').classList.remove('show');
    },

    // 更新 body class
    updateBodyClass() {
      if (this.isDarkMode) {
        document.body.classList.add("dark-mode");
      } else {
        document.body.classList.remove("dark-mode");
      }
    },

    // 更新 PWA 颜色
    PWAColor() {
      if (this.isDarkMode) {
        document
          .querySelector('meta[name="theme-color"]')
          .setAttribute("content", "#171a1d");
        document
          .querySelector('meta[name="background-color"]')
          .setAttribute("content", "#212529");
      } else {
        document
          .querySelector('meta[name="theme-color"]')
          .setAttribute("content", "#f8f9fa");
        document
          .querySelector('meta[name="background-color"]')
          .setAttribute("content", "#ffffff");
      }
    },

    // 点击 Logo 事件处理
    handleLogoClick() {
      if (window.scrollY === 0) {
        this.$store.commit('setRefreshEveryThing', true);
      }
      this.$trackEvent('Nav', 'NavClick', 'Logo');
    },
  },
  mounted() {
    this.$store.dispatch('checkDarkMode');
    this.updateBodyClass();
    this.PWAColor();
    setTimeout(() => {
      this.getGitHubStars();
    }, 1000)
  },
}
</script>

<style scoped>
/*==================== Dark Light Button Implementation ====================*/
.jn-checkbox {
  display: none;
}

.slide-fade-enter-active {
  transition: all 0.3s ease-in-out;
}

.slide-fade-leave-active {
  transition: all 0.3s cubic-bezier(1, 0.5, 0.8, 1);
}

.slide-fade-enter-from {
  transform: translateY(30px);
  opacity: 0;
}

.slide-fade-leave-to {
  transform: translateY(-30px);
  opacity: 0;
}

.jn-fs {
  font-size: smaller;
  display: flex;
  max-height: 25pt;
  overflow: hidden;
  width: fit-content;
}

.jn-w {
  width: 60pt;
}

.redstar {
  color: rgb(253 131 3);
}

.yellowstar {
  color: rgb(255 216 0);
}

.switch {
  background-color: #111;
  border-radius: 50px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 5px;
  position: relative;
  height: 26px;
  width: 50px;
  transform: scale(0.7);
  box-shadow: 0 0 2px white;
}

.switch .ball {
  background-color: #fff;
  border-radius: 50%;
  position: absolute;
  top: 2px;
  left: 2px;
  height: 22px;
  width: 22px;
  transform: translateX(0px);
  transition: transform 0.2s linear;
}

.jn-checkbox:checked+.switch .ball {
  transform: translateX(24px);
}

.jn-button:hover {
  border: 0;
}

.jn-button:active {
  border: 0;
}

.jn-button:focus {
  border: 0;
}

.jn-button {
  border: 0;
}

.background-animation-light {
  position: relative;
  overflow: hidden;
  display: inline-flex;
}

.background-animation-light::before {
  content: '';
  position: absolute;
  bottom: 0;
  left: -100%;
  width: 100%;
  height: 10%;
  background-color: rgb(0, 0, 0);
  animation: backgroundSlide 1s linear infinite;
}

.background-animation-dark {
  position: relative;
  overflow: hidden;
  display: inline-flex;
}

.background-animation-dark::before {
  content: '';
  position: absolute;
  bottom: 0;
  left: -100%;
  width: 100%;
  height: 10%;
  background-color: rgb(255, 255, 255);
  animation: backgroundSlide 1s linear infinite;
}

@keyframes backgroundSlide {
  from {
    left: -100%;
  }

  to {
    left: 100%;
  }
}
</style>
