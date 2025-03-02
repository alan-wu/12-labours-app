<template>
  <div class="header">
    <div class="header-container">
      <button class="nav__mobile-menu" @click="openMobileNav">
        <svgicon name="hamburger" height="40" width="40"/>
      </button>
      <div class="logo">
        <component :is="linkComponent" :to="{ name: 'index' }">
          <!-- Provide a way to slot in the logo -->
          <slot name="logo">
            <twelve-labours-logo/>
          </slot>
        </component>
      </div>
      <div :class="[menuOpen ? 'overlay' : '']">
        <div class="navigation" :class="[menuOpen ? 'open' : '']">
          <div class="menu-close">
            <div class="logo-sm">
              <component :is="linkComponent" :to="{ name: 'index' }">
                <slot name="logo">
                  <twelve-labours-logo/>
                </slot>
              </component>
            </div>
            <div class="close">
              <button @click="openMobileNav">
                <svgicon name="close" height="30" width="30" />
              </button>
            </div>
          </div>
          <ul>
            <li v-for="link in links" :key="link.href" style="z-index: 100;">
              <component
                :is="linkComponent"
                :to="link.href"
                :class="{ active: activeLink(link.href) }"
                exact-active-class="active">
                  {{ link.displayTitle.toUpperCase() }}
                </component>
            </li>
          </ul>
          <!-- //undo later::
          <div class="menu-footer">
            <footer-links menuOnly/>
          </div>-->
        </div>
      </div>
      <div v-if="$auth.loggedIn && $auth.strategy.token.status().valid()" class="login vertical-flex"> 
        <span id="welcome">Welcome {{$auth.user.first_name}} {{$auth.user.last_name}}</span>
        <component :is="linkComponent" to="/profile">
          <el-button>Account</el-button>
        </component>
        <component :is="linkComponent" @click.native="signout" to="/">
          Log out
        </component>
      </div>
      <div v-else  class="login vertical-flex"> 
        <component :is="linkComponent" to="/login">
          <el-button>Login</el-button>
        </component>
        <component :is="linkComponent" to="/signup">
          Sign up Here
        </component>
      </div>
    </div>
  </div>
</template>

<script>
//undo later:: import TwelveLaboursLogo from "../../TwelveLaboursLogo/src/TwelveLaboursLogo.vue";
//undo later:: import FooterLinks from "../../FooterLinks/src/FooterLinks.vue";

export default {
  name: "TwelveLaboursHeader",
  props: {
    linkComponent: {
      type: String,
      default: "nuxt-link"
    },
    links: {
      type: Array,
      default: function() { return [
          {
            title: "data-and-models",
            displayTitle: "Data & Models",
            href: "/data?type=dataset"
          },
          {
            title: "resources",
            displayTitle: "Resources",
            href: "/resources"
          },
          {
            title: "about",
            displayTitle: "About",
            href: `/about`
          },
          {
            title: "news-and-events",
            displayTitle: "News & Events",
            href: "/news-and-events"
          },
          {
            title: "search",
            displayTitle: "Search",
            href: "/search"
          }
        ]
      }
    },
  
    currentPath: {
      type: String,
      default: "/"
    }
  },
  components: {
    //undo later:: TwelveLaboursLogo,
    //undo later:: FooterLinks
  },
  data: () => ({
    menuOpen: false,
    searchOpen: false,
    searchQuery: "",
    searchSelect: "data",
    searchSelectOptions: [
      {
        key: "data",
        value: "data",
        label: "Datasets"
      },
      {
        key: "resources",
        value: "resources",
        label: "Resources"
      },
      {
        key: "news-and-events",
        value: "news-and-events",
        label: "News and Events"
      },
      {
        key: "help",
        value: "help",
        label: "Support Center"
      }
    ]
  }),

  computed: {

    /**
     * Compute if search should be visible
     * @returns {Boolean}
     */
    shouldShowSearch: function() {
      if (this.$route)
        return this.$route.name !== "data";
      else
        return true
    }
  },

  watch: {
    /**
     * Watches for the route path to hide
     * mobile nav on menu click
     **/
    currentPath: {
      handler: function(val) {
        if (val) {
          this.menuOpen = false;
        }
      },
      immediate: true
    },

    /**
     * Watches menuOpen to check if it's false
     * to enable scrolling
     */
    menuOpen: {
      handler: function(val) {
        if (!val) {
          this.$emit("updateDisabledScrolling", false);
        }
      },
      immediate: true
    }
  },

  methods: {
    /* Signs out of current strategy */
    signout:async function(){
      await this.$auth.logout().then(() => {
        this.$toast.success('Logged out of 12 Labours',{duration:3000, position: 'bottom-right'}) 
      })
    },

    /**
     * Sets a link to active based on current page
     * @param {String} query
     */
    activeLink: function(query) {
      if (this.currentPath === query) {
        return true;
      } else {
        return false;
      }
    },
    /**
     * Opens the mobile version of the navigation
     */
    openMobileNav: function() {
      if (!this.menuOpen) {
        this.searchOpen = false; // just in case the search menu is open aconstso
        this.$emit("updateDisabledScrolling", false);
        this.menuOpen = true;
      } else {
        this.menuOpen = false;
        this.$emit("updateDisabledScrolling", false);
      }
    },

  
    /**
     * Opens the mobile version of the search bar
     */
    openMobileSearch: function() {
      this.searchOpen = true;
      this.menuOpen = false;
      this.$emit("updateDisabledScrolling", false);
    },

    /**
     * Closes the mobile version of the search bar
     */
    closeMobileSearch: function() {
      this.searchOpen = false;
      this.$emit("updateDisabledScrolling", false);
    },

    /**
     * Executes a search query based on selected
     * option and query
     */

    executeSearch: function() {
      const option = this.searchSelectOptions.find(
        o => o.value === this.searchSelect
      );
      const searchKey = option.value === "data" ? "q" : "search";
      const type =
        option.value === "data"
          ? "dataset"
          : option.value === "resources"
          ? "sparcPartners"
          : undefined;

      this.$router.push({
        name: option.value,
        query: {
          type,
          [searchKey]: this.searchQuery
        }
      });

      this.searchQuery = "";
      this.searchSelect = "data";
    }
  }
};
</script>

<style scoped lang="scss">
//undo later:: @import "@/assets/_variables.scss";

.header{
  display: flex;    
}


.header-container {
  display: flex;
  width: 100%;
  background-color: $cochlear;
  height:6.25rem;
  padding:0 3.75rem;
  column-gap:2rem;
  align-items: center;
  @media only screen and (max-width: $viewport-md){
    padding:0 2rem;
  }   
}

.logo
{
  display:flex;
  align-items:center;
  height: 6.25rem;   
  width: 7.3rem;  
  white-space: nowrap;
  @media only screen and (max-width: $viewport-md){
    display:none;
  }
}

.navigation {
  
  ul {
    padding:0;
    margin:0 0 0.5rem 0;
    display: flex;
    align-items:center;
    column-gap:3rem;

    li {
      display: inline;
      a {
        text-decoration: none  !important;
        font: normal normal normal 1rem/1.13rem $font-family;
        color:$app-primary-color !important; 
        &.active,&:hover,&:focus {
          color: $app-primary-color;
        }  
        @media only screen and (max-width: $viewport-md){
          color:$cochlear !important;
        } 
      }
    }
  }

  @media only screen and (max-width: $viewport-md) {
    & {
      background: $mildBlue;
      display: none;     
      flex-direction: column; 
      bottom: 0;
      left: 0;
      top:0;
      position: fixed;
      z-index: 9998;

      &.open {
        display: flex;
        width: 15rem;
        overflow: auto;
      }
    }
    ul {
      flex-direction: column;
      row-gap:0.5rem;
      height: calc(100% - 210px);
      overflow-y: auto;

      //For Firefox
      /*scrollbar-width: thin;
      scrollbar-color: $mildBlue $cochlear;*/
    }
  }
}  

.login{
  margin-left: auto;
  text-align:center;
  span#welcome{
    font-size:0.8rem;
    font-style: italic;
  }
}

.nav__mobile-menu {
  background: none;
  border: none;
  display: none;
  margin: 0px;
  outline: none;
  //transform: translate(0.75rem, -0.5rem);
  -webkit-appearance: none;
  position:fixed;
  @media only screen and (max-width: $viewport-md) {
    & {
      display: block;
    }
  }
}

.menu-close{
  display:none;
  @media only screen and (max-width: $viewport-md) {
    display:flex;
    justify-content:space-between;
  }
}

.logo-sm
{
  display:none;
  @media only screen and (max-width:$viewport-md){
    display:block;
    height: 3rem;   
    width: 3rem;  
    white-space: nowrap;
    padding:0.25rem
  }
}

.menu-footer{
  display:none;
  @media only screen and (max-width: $viewport-md) {
    display:block;
    width:100%;
    position:absolute;
    bottom:0;
    z-index: 9999;
  }
}

</style>

