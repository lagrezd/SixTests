<template>
  <div class="app-main">
    <!-- App Navigation -->
    <header>
      <nav>
      <div class="logo">
        <div class="responsive-wrapper">
        <img src="/build/img/logo_new_icon.png" />
        </div>
      </div>
      <ul class="nav-items">
        <li>
        <a v-link="{ path: '/app/exams' }">
          <div class="navicon">
            <i class="material-icons">dashboard</i>
          </div>
          <div class="navtext">
            Exams
          </div>
        </a>
        </li>
      </ul>
      </nav>
    </header>
    <!-- Main App Display -->
    <section class="main-app-display">
      <div id="exam-notifications" class="mdl-js-snackbar mdl-snackbar">
        <div class="mdl-snackbar__text"></div>
        <button class="mdl-snackbar__action" type="button"></button>
      </div>
      <router-view></router-view>
    </section>
  </div>
</template>