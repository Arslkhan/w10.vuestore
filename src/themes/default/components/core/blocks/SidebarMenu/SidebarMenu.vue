<template>
  <div class="sidebar-menu fixed mw-100">
    <div class="row brdr-bottom-1 brdr-cl-bg-secondary">
      <div
        v-if="submenu.depth"
        class="col-xs"
      >
        <sub-btn type="back" class="bg-cl-transparent brdr-none" />
      </div>
      <div class="col-xs">
        <button
          type="button"
          :aria-label="$t('Close')"
          class="w-100 inline-flex end-xs bg-cl-transparent brdr-none p0 close-btn"
          @click="closeMenu"
        >
          <i class="material-icons p15">close</i>
        </button>
      </div>
    </div>
    <div class="sidebar-menu__container row" ref="container">
      <div class="col-xs-12 h4 serif">
        <ul class="p0 m0 relative sidebar-menu__list" :style="mainListStyles">
          <li
            @click="closeMenu"
            class="brdr-bottom-1 brdr-cl-bg-secondary "
          >
            <router-link
              class="block px25 py20 cl-accent no-underline"
              :to="localizedRoute('/')"
              exact
            >
              {{ $t('Home') }}
            </router-link>
          </li>
          <li
            class="brdr-bottom-1 brdr-cl-bg-secondary flex hidden-xs"
            :key="category.slug"
            @click="closeMenu"
            v-for="category in visibleCategories"
          >
            <div
              v-if="isCurrentMenuShowed"
              class="subcategory-item"
            >
              <sub-btn
                v-if="category.children_count > 0"
                class="bg-cl-transparent brdr-none fs-medium"
                :id="category.id"
                :name="category.name"
              />
              <router-link
                v-else
                class="px25 py20 cl-accent no-underline col-xs"
                :to="categoryLink(category)"
              >
                {{ category.name }}
              </router-link>
            </div>

            <sub-category
              :category-links="category.children_data"
              :id="category.id"
              :parent-slug="category.slug"
              :parent-path="category.url_path"
            />
          </li>
          
            <li class="brdr-bottom-1 brdr-cl-bg-secondary flex">

             <router-link
              class="px25 py20 cl-accent no-underline col-xs"
              :to="localizedRoute('/our-range')"
              exact
              @click.native="closeMenu"
            >
              Our Range
            </router-link>

            </li>
            <li class="brdr-bottom-1 brdr-cl-bg-secondary flex">

             <router-link
              class="px25 py20 cl-accent no-underline col-xs"
              :to="localizedRoute('/innovation')"
              exact
              @click.native="closeMenu"
            >
              Innovation
            </router-link>

             </li>
             <li class="brdr-bottom-1 brdr-cl-bg-secondary flex">
          <router-link
              class="px25 py20 cl-accent no-underline col-xs"
              :to="localizedRoute('/about-us')"
              exact
              @click.native="closeMenu"
            >
              About us
            </router-link>
          </li>
             <li class="brdr-bottom-1 brdr-cl-bg-secondary flex">

             <router-link
              class="px25 py20 cl-accent no-underline col-xs"
              :to="localizedRoute('/contact-us')"
              exact
              @click.native="closeMenu"
            >
              Contact Us
            </router-link>
            </li>
          <li
            @click="login"
            class="brdr-bottom-1 brdr-cl-secondary flex"
          >
            <sub-btn
              v-if="currentUser"
              id="my-account-links"
              :is-category="false"
              :name="$t('My Account')"
              class="px25 py20 cl-accent no-underline col-xs"
            />
            <sub-category
              v-if="currentUser"
              :my-account-links="myAccountLinks"
              id="my-account-links"
              @click.native="closeMenu"
            />
            <a
              v-if="!currentUser && isCurrentMenuShowed"
              href="#"
              @click.prevent="closeMenu"
              class="px25 py20 cl-accent no-underline col-xs"
            >
              {{ $t('My Account') }}
            </a>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import { mapState } from 'vuex'
import i18n from '@vue-storefront/i18n'
import SidebarMenu from '@vue-storefront/core/compatibility/components/blocks/SidebarMenu/SidebarMenu'
import SubBtn from 'theme/components/core/blocks/SidebarMenu/SubBtn'
import SubCategory from 'theme/components/core/blocks/SidebarMenu/SubCategory'
import { formatCategoryLink } from '@vue-storefront/core/modules/url/helpers'
import { disableBodyScroll, clearAllBodyScrollLocks } from 'body-scroll-lock'

export default {
  components: {
    SubCategory,
    SubBtn
  },
  mixins: [SidebarMenu],
  data () {
    return {
      myAccountLinks: [
        {
          id: 1,
          name: i18n.t('My profile'),
          url: '/my-account'
        },
        {
          id: 2,
          name: i18n.t('My shipping details'),
          url: '/my-account/shipping-details'
        },
        
        {
          id: 4,
          name: i18n.t('My orders'),
          url: '/my-account/orders'
        }
        
      ],
      componentLoaded: false
    }
  },
  computed: {
    mainListStyles () {
      return this.submenu.depth ? `transform: translateX(${this.submenu.depth * 100}%)` : false
    },
    ...mapState({
      submenu: state => state.ui.submenu,
      currentUser: state => state.user.current
    }),
    getSubmenu () {
      return this.submenu
    },
    visibleCategories () {
      return this.categories.filter(category => {
        return category.product_count > 0 || category.children_count > 0
      })
    },
    isCurrentMenuShowed () {
      return !this.getSubmenu || !this.getSubmenu.depth
    }
  },
  mounted () {
    this.$nextTick(() => {
      this.componentLoaded = true;
      disableBodyScroll(this.$refs.container)
    })
  },
  destroyed () {
    clearAllBodyScrollLocks()
  },
  methods: {
    login () {
      if (!this.currentUser && this.isCurrentMenuShowed) {
        this.$nextTick(() => {
          this.$store.commit('ui/setAuthElem', 'login')
          this.$bus.$emit('modal-show', 'modal-signup')
          this.$router.push({ name: 'my-account' })
        })
      }
    },
    categoryLink (category) {
      return formatCategoryLink(category)
    }
  }
}
</script>

<style lang="scss" scoped>
@import "~theme/css/animations/transitions";
@import '~theme/css/variables/colors';
@import '~theme/css/helpers/functions/color';
$bg-secondary: color(secondary, $colors-background);
$color-gainsboro: color(gainsboro);
$color-matterhorn: color(matterhorn);
$color-mine-shaft: color(mine-shaft);

.sidebar-menu {
  height: 100vh;
  width: 350px;
  overflow: hidden;

  @media (max-width: 767px) {
    width: 100vh;
  }

  &__container {
    overflow-y: auto;
    -webkit-overflow-scrolling: touch;
    height: calc(100% - 55px);
  }

  &__list {
    transition: transform $duration-main $motion-main;
  }

  ul {
    list-style-type: none;
  }

  li {
    &:hover,
    &:focus {
      background-color: $color-gainsboro;
    }
    &.bg-cl-primary {
      &:hover,
      &:focus {
        background-color: $bg-secondary;
      }
    }
    a {
      color: $color-mine-shaft;
    }
  }

  .subcategory-item {
    display: flex;
    width: 100%;
  }

  button {
    color: $color-mine-shaft;a {
      color: $color-mine-shaft;
    }
  }

  .close-btn {
    i {
      color: $color-gainsboro;
    }
    &:hover,
    &:focus {
      i {
        color: $color-matterhorn;
      }
    }
  }

}
</style>
