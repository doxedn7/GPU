<template>
  <v-app>
    <v-navigation-drawer width="250" dark app v-model="sideNav">
      <v-list>
        <v-list-tile v-if="userIsAuthenticated==false">
          <v-list-tile-content>
            <v-dialog width="500" v-model="dialog">
              <template v-slot:activator="{ on }">
                <v-btn v-on="on" flat>
                  <v-icon left>fas fa-user-circle</v-icon>
                  <span>Se Connecter</span>
                </v-btn>
              </template>
              <v-card
                style="border:5px solid #008080;border-radius:20px;-moz-border-radius:20px;-webkit-border-radius:20px;background-color:#424242"
              >
                <v-form @submit.prevent="onSignIn()">
                  <v-layout column align-center>
                    <img
                      src="https://firebasestorage.googleapis.com/v0/b/gpufinal.appspot.com/o/logo.png?alt=media&token=3bb68f47-2e5d-4a41-9844-22ad4f199fd5"
                      width="200"
                      height="125"
                    >
                    <span style="font-size:18px;color:#F5DCD7">Identifiez-vous</span>
                  </v-layout>
                  <v-flex mx-5 mt-3 justify-center>
                    <v-text-field
                      dark
                      label="Email"
                      v-model="email"
                      prepend-inner-icon="fas fa-user"
                      color="#F5DCD7"
                    ></v-text-field>
                  </v-flex>
                  <v-flex mx-5 mt-3 justify-center>
                    <v-text-field
                      dark
                      type="password"
                      v-model="password"
                      label="Mot de passe"
                      prepend-inner-icon="fas fa-unlock-alt"
                      color="#F5DCD7"
                    ></v-text-field>
                  </v-flex>
                  <v-flex mx-5 mt-3 justify-center>
                    <router-link
                      to="/signup"
                      @click.native="dialog=false"
                    >Pas de compte ? Créer-en-un</router-link>
                  </v-flex>
                  <v-layout column wrap align-center justify-center mx-5>
                    <v-flex>
                      <v-btn type="submit" color="#F5DCD7">
                        <div>Se Connecter</div>
                      </v-btn>
                    </v-flex>
                  </v-layout>
                </v-form>
              </v-card>
            </v-dialog>
          </v-list-tile-content>
        </v-list-tile>

        <v-list-group v-else>
          <v-list-tile slot="activator">
            <v-list-tile-action>
              <v-avatar>
                <v-img
                  :src="avatar"
                ></v-img>
              </v-avatar>
            </v-list-tile-action>
            <v-list-tile-title>{{username}}</v-list-tile-title>
          </v-list-tile>
          <v-list-tile to="/settings">
            <v-list-tile-title>Paramètres</v-list-tile-title>
          </v-list-tile>
          <v-list-tile :to="/user/ +this.$store.state.user.id">
            <v-list-tile-title>Mon profil</v-list-tile-title>
          </v-list-tile>
          <v-list-tile v-if="this.$store.state.user.isAdmin == true" to="/suggested_games">
            <v-list-tile-title>Jeux suggérés</v-list-tile-title>
          </v-list-tile>
          <v-list-tile v-if="this.$store.state.user.isAdmin == true" to="/suggested_articles">
            <v-list-tile-title>Articles suggérés</v-list-tile-title>
          </v-list-tile>
          <v-list-tile @click="onLogout()">
            <v-list-tile-title>Déconnexion</v-list-tile-title>
          </v-list-tile>
        </v-list-group>

        <v-list-tile>
          <v-autocomplete
            name="name"
            :items="gamesCharged"
            :search-input.sync="search"
            v-model="selectedGame"
            label="Rechercher"
            height="40px"
            color="#008080"
            @change="getSelectedGame()"
            item-text="nom"
            return-object
          ></v-autocomplete>
        </v-list-tile>
        <v-list-group>
          <v-list-tile slot="activator">
            <v-list-tile-action>
              <v-icon>fas fa-gamepad</v-icon>
            </v-list-tile-action>
            <v-list-tile-content>
              <v-list-tile-title>Jeux</v-list-tile-title>
            </v-list-tile-content>
          </v-list-tile>

          <v-list-tile v-for="(item, index) in jeux" :key="index" @click.stop :to="item.link">
            <v-list-tile-title>{{item.title}}</v-list-tile-title>
          </v-list-tile>
        </v-list-group>

        <v-list-tile to="News">
          <v-list-tile-action>
            <v-icon>far fa-newspaper</v-icon>
          </v-list-tile-action>
          <v-list-tile-title>News</v-list-tile-title>
        </v-list-tile>

        <v-list-tile v-for="(item, index) in console" :key="index" @click.stop :to="item.link">
          <v-list-tile-action>
            <v-icon>{{item.icon}}</v-icon>
          </v-list-tile-action>
          <v-list-tile-title>{{item.title}}</v-list-tile-title>
        </v-list-tile>
      </v-list>
    </v-navigation-drawer>

    <v-toolbar app color="#D32F2F" id="youth" dark absolute v-model="onScroll">
      <v-toolbar-side-icon @click.stop="sideNav = !sideNav" class="hidden-md-and-up"></v-toolbar-side-icon>

      <v-toolbar-items>
        <v-btn to="/" color="transparent" flat width="120" height="120">
          <img
            src="https://firebasestorage.googleapis.com/v0/b/gpufinal.appspot.com/o/logo.png?alt=media&token=3bb68f47-2e5d-4a41-9844-22ad4f199fd5"
            width="220"
            height="120"
          >
        </v-btn>
      </v-toolbar-items>
      <v-toolbar-items class="hidden-sm-and-down">
        <v-menu open-on-hover bottom offset-y origin="center center" transition="scale-transition">
          <v-btn slot="activator" flat :to="{ path: '/Game' }">Jeux</v-btn>
          <v-list>
            <v-list-tile v-for="(item, index) in jeux" :key="index" @click.stop :to="item.link">
              <v-list-tile-title>{{item.title}}</v-list-tile-title>
            </v-list-tile>
          </v-list>
        </v-menu>
        <v-btn slot="activator" flat :to="{ path: '/News' }">News</v-btn>
      </v-toolbar-items>
      <v-toolbar-items class="hidden-sm-and-down">
        <v-btn
          v-for="(icone, index) in console"
          :key="index"
          class="mx-2"
          dark
          icon
          :to="icone.link"
        >
          <v-icon size="24px">{{ icone.icon }}</v-icon>
        </v-btn>
      </v-toolbar-items>

      <v-spacer></v-spacer>

      <v-toolbar-items class="hidden-sm-and-down">
        <v-autocomplete
          name="name"
          :items="gamesCharged"
          :search-input.sync="search"
          v-model="selectedGame"
          label="Rechercher"
          height="40px"
          v-if="searchVisible"
          color="#008080"
          @change="getSelectedGame()"
          item-text="nom"
          return-object
        ></v-autocomplete>
        <v-layout align-center>
          <v-btn icon @click="searchVisible = !searchVisible">
            <v-icon>fas fa-search</v-icon>
          </v-btn>

          <v-flex>
            <v-dialog v-if="this.userIsAuthenticated==false" width="500" v-model="dialog">
              <template v-slot:activator="{ on }">
                <v-btn v-on="on" flat>
                  <v-icon left>fas fa-user-circle</v-icon>
                  <span>Se Connecter</span>
                </v-btn>
              </template>

              <v-card
                style="border:5px solid #008080;border-radius:20px;-moz-border-radius:20px;-webkit-border-radius:20px;background-color:#424242"
              >
                <v-form @submit.prevent="onSignIn()">
                  <v-layout column align-center>
                    <img
                      src="https://firebasestorage.googleapis.com/v0/b/gpufinal.appspot.com/o/logo.png?alt=media&token=3bb68f47-2e5d-4a41-9844-22ad4f199fd5"
                      width="200"
                      height="125"
                    >
                    <span style="font-size:18px;color:#F5DCD7">Identifiez-vous</span>
                  </v-layout>
                  <v-flex mx-5 mt-3 justify-center>
                    <v-text-field
                      dark
                      label="Email"
                      v-model="email"
                      prepend-inner-icon="fas fa-user"
                      color="#F5DCD7"
                    ></v-text-field>
                  </v-flex>
                  <v-flex mx-5 mt-3 justify-center>
                    <v-text-field
                      dark
                      type="password"
                      v-model="password"
                      label="Mot de passe"
                      prepend-inner-icon="fas fa-unlock-alt"
                      color="#F5DCD7"
                    ></v-text-field>
                  </v-flex>
                  <v-flex mx-5 mt-3 justify-center>
                    <router-link
                      to="/signup"
                      @click.native="dialog=false"
                    >Pas de compte ? Créer-en-un</router-link>
                  </v-flex>
                  <v-layout column wrap align-center justify-center mx-5>
                    <v-flex>
                      <v-btn type="submit" color="#F5DCD7">
                        <div>Se Connecter</div>
                      </v-btn>
                    </v-flex>

                  </v-layout>
                </v-form>
              </v-card>
            </v-dialog>

            <v-menu
              v-else
              open-on-hover
              bottom
              offset-y
              origin="center center"
              transition="scale-transition"
            >
              <v-btn slot="activator" flat>
                <v-avatar>
                  <v-img
                    :src="avatar"
                  ></v-img>
                </v-avatar>&nbsp;{{username}}
              </v-btn>
              <v-list>
                <v-list-tile to="/settings">
                  <v-list-tile-title>Paramètres</v-list-tile-title>
                </v-list-tile>
                <v-list-tile :to="/user/ +this.$store.state.user.id">
                  <v-list-tile-title>Mon profil</v-list-tile-title>
                </v-list-tile>
                <v-list-tile v-if="this.$store.state.user.isAdmin == true" to="/suggested_games">
                  <v-list-tile-title>Jeux suggérés</v-list-tile-title>
                </v-list-tile>
                <v-list-tile v-if="this.$store.state.user.isAdmin == true" to="/suggested_articles">
                  <v-list-tile-title>Articles suggérés</v-list-tile-title>
                </v-list-tile>
                <v-list-tile @click="onLogout()">
                  <v-list-tile-title>Déconnexion</v-list-tile-title>
                </v-list-tile>
              </v-list>
            </v-menu>
          </v-flex>
        </v-layout>
      </v-toolbar-items>
    </v-toolbar>

    <v-layout column fill-height>
      <v-parallax
        src="https://firebasestorage.googleapis.com/v0/b/gpufinal.appspot.com/o/background.png?alt=media&token=2f5715e8-a52a-4343-98f9-83e1193e8b75"
        id="paral"
        height="7000"
      >
        <v-flex xl12 xs12 md12 fluid fill-height>
          <v-content>
            <router-view :key="$route.path"></router-view>
          </v-content>
        </v-flex>
      </v-parallax>

      <v-flex>
        <v-footer dark height="auto" id="footer">
          <v-card class="flex" flat tile>
            <v-card-title class="grey darken-3">
              <v-layout align-center justify-space-around row fill-height>
                <v-flex xs12>
                  <strong class="subheading font-italic bold font-weight-medium">Game Players Union</strong>
                </v-flex>

                <v-flex xs11>
                  <div>
                    <img
                      src="https://cdn.discordapp.com/attachments/521706386829082626/540180308390314004/Logo.png"
                      width="150"
                      height="75"
                    >
                  </div>
                </v-flex>

                <v-flex xl3>
                  <v-btn
                    target="_blank"
                    v-for="(icone, index) in social"
                    :key="index"
                    class="mx-3"
                    dark
                    icon
                    :href="icone.link"
                  >
                    <v-icon size="24px">{{ icone.icon }}</v-icon>
                  </v-btn>
                </v-flex>
              </v-layout>
            </v-card-title>
            <v-layout align-center justify-space-around row fill-height>
              <v-btn to="/about" flat round>
                <span>A propos</span>
              </v-btn>
            </v-layout>
            <v-card-actions class="grey darken-4 justify-center">
              &copy;2019 —
              <strong primary>&nbsp;GPU Team</strong>
            </v-card-actions>
          </v-card>
        </v-footer>
      </v-flex>
    </v-layout>

    <v-btn id="myBtn" @click="topFunction" ripple icon>
      <v-icon>fa-chevron-circle-up</v-icon>
    </v-btn>
  </v-app>
</template>

<script>
import HelloWorld from "./components/HelloWorld";
import firebase from "firebase/app";
import "firebase/auth";
import { store } from "./store";
import { isString } from "util";

export default {
  name: "App",
  components: {
    HelloWorld,
  },
  data() {
    return {
      email: "",
      password: "",
      confirmPassword: "",
      username: "",
      avatar: null,
      selectedGame: null,
      search: "something",
      dialog: false,
      sideNav: false,
      icons: ["fab fa-facebook", "fab fa-twitter", "fab fa-instagram"],
      isVisible: true,
      searchVisible: false,
      search: null,
      model: null,
      social: [
        {
          icon: " fab fa-facebook",
          link: "https://www.facebook.com/GamePlayerUnion/"
        },
        {
          icon: "fab fa-instagram",
          link: "https://www.instagram.com/gameplayerunion/"
        },
        {
          icon: "fab fa-twitter",
          link: "https://twitter.com/GamePlayerUnion?s=09"
        }
      ],
      console: [
        {
          icon: "fab fa-playstation",
          link: "/PlaystationGames",
          title: "Playstation"
        },
        {
          icon: "fab fa-xbox",
          link: "/XboxGames",
          title: "Xbox"
        },
        {
          icon: "fab fa-nintendo-switch",
          link: "/SwitchGames",
          title: "Nintendo Switch"
        },
        {
          icon: "fas fa-desktop",
          link: "/PcGames",
          title: "PC"
        }
      ],
      jeux: [
        {
          title: "Classement",
          link: "/GameRank"
        },
        {
          title: "Tous les jeux",
          link: "/AllGames"
        },
        {
          title: "Nouveautés",
          link: "/RecentGames"
        }
      ],
      profil: [
        {
          title: "Paramètres",
          link: "/settings"
        },
        {
          title: "Déconnexion",
          link: ""
        }
      ]
    };
  },
  watch: {
      username: function() {
        this.username = this.$store.state.user.pseudo
      }
  },
  computed: {
    onScroll() {
      window.onscroll = () => {
        this.scrollFunction();
      };
    },
    userIsAuthenticated() {
      if(
        this.$store.getters.user !== null &&
        this.$store.getters.user !== undefined
      )
      {
        this.username = this.$store.state.user.pseudo
        if(this.$store.state.user.image !== null && this.$store.state.user.image !== undefined && this.$store.state.user.image.toString().localeCompare("") !=0) 
        {
          this.avatar = this.$store.state.user.image
        }
        else
        {
          this.avatar = "https://firebasestorage.googleapis.com/v0/b/gpufinal.appspot.com/o/Portrait_placeholder.png?alt=media&token=49580b44-9483-4418-8c35-92fcd766e72d"
        }
      
      }
      return (
        this.$store.getters.user !== null &&
        this.$store.getters.user !== undefined
      );
    },
    gamesCharged: function() {
      return this.$store.state.loadedAllGames;
    }
  },
  mounted: function() {
    this.getHeight();
  },
  methods: { 
    onSignIn() {
      if (this.sideNav == true) {
        this.sideNav = !this.sideNav;
      }
      this.$store.dispatch("signUserIn", {
        email: this.email,
        password: this.password
      });
    },
    scrollFunction() {
      var limit =
        Math.max(
          document.body.scrollHeight,
          document.body.offsetHeight,
          document.documentElement.clientHeight,
          document.documentElement.scrollHeight,
          document.documentElement.offsetHeight
        ) - window.innerHeight;
      if (
        (document.body.scrollTop >
          document.getElementById("youth").clientHeight &&
          document.body.scrollTop <=
            limit - document.getElementById("footer").clientHeight) ||
        (document.documentElement.scrollTop >
          document.getElementById("youth").clientHeight &&
          document.documentElement.scrollTop <=
            limit - document.getElementById("footer").clientHeight)
      ) {
        document.getElementById("myBtn").style.bottom = "30px";
        document.getElementById("myBtn").style.display = "block";
        document.getElementById("myBtn").style.position = "fixed";
      } else if (
        document.body.scrollTop >
          limit - document.getElementById("footer").clientHeight ||
        document.documentElement.scrollTop >
          limit - document.getElementById("footer").clientHeight
      ) {
        document.getElementById("myBtn").style.bottom =
          document.getElementById("footer").clientHeight.toString() + "px";
        document.getElementById("myBtn").style.position = "absolute";
      } else {
        document.getElementById("myBtn").style.display = "none";
        document.getElementById("myBtn").style.bottom = "30px";
      }
    },
    topFunction() {
      window.scrollTo({
        top: 0,
        left: 0,
        behavior: "smooth"
      });
      document.getElementById("myBtn").style.display = "none";
    },
    getHeight() {
      if (
        document.documentElement.scrollHeight >
          document.documentElement.clientHeight ||
        document.body.scrollHeight > document.body.clientHeight
      ) {
        document.getElementById("paral").style.height = null;
        document.getElementById("paral").style.height = "150%";
      } else {
        document.getElementById("paral").style.height = null;
        document.getElementById("paral").style.height = "100%";
      }
    },
    openForm() {
      document.getElementById("myForm").style.display = "block";
      this.setVisible;
    },
    closeForm() {
      document.getElementById("myForm").style.display = "none";
      this.setVisible();
    },
    setVisible() {
      this.isVisible == false
        ? (this.isVisible = true)
        : (this.isVisible = false);
    },
    onLogout() {
      this.dialog = false;
      this.$store.dispatch("logoutUser");
    },
    getSelectedGame() {
      let context = this;
      setTimeout(() => {
        if (
          this.selectedGame.plateformeJeux.localeCompare("PC", "en", {
            sensitivity: "base"
          }) == 0
        ) {
          this.$router.push({
            name: "gameDescPC",
            params: { id: this.selectedGame.id, force: true }
          });
          this.$forceUpdate();
        }
        if (
          this.selectedGame.plateformeJeux.localeCompare("PS", "en", {
            sensitivity: "base"
          }) == 0
        ) {
          this.$router.push({
            name: "gameDescPS",
            params: { id: this.selectedGame.id, force: true }
          });
          this.$forceUpdate();
        }
        if (
          this.selectedGame.plateformeJeux.localeCompare("XBOX", "en", {
            sensitivity: "base"
          }) == 0
        ) {
          this.$router.push({
            name: "gameDescXBOX",
            params: { id: this.selectedGame.id },
            force: true
          });
          this.$forceUpdate();
        }
        if (
          this.selectedGame.plateformeJeux.localeCompare("SWITCH", "en", {
            sensitivity: "base"
          }) == 0
        ) {
          this.$router.push({
            name: "gameDescSWITCH",
            params: { id: this.selectedGame.id }
          });
          this.$forceUpdate();
        }
        this.$nextTick(() => {
          this.selectedGame = null;
        });
      }, 0);
    }
  }
};
</script>

<style src="./css/homepage.css">
</style>
