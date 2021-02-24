<template>
  <div class="o-search-panel">
    <div
      v-if="noResultsMessage"
      class="no-results"
    >
      {{ $t(noResultsMessage) }}
    </div>

    <div v-else class="container">
      <div class="categories">
        <SfHeading :level="3" :title="$t('Categories')" class="categories__title sf-heading--left" />
        <SfList v-if="visibleProducts.length && parent.length > 1" class="categories__listing">
          <div v-for="(category,n) in parent" :key="category.id" :class="{'selected': isCategorySelected(category)}">
            <label :v-html="category.link" style="font-size:20px;"><input type="checkbox"
                                                                          v-model="children[n]" :id="category.id" @change="check($event,category.items)"
            ><span>{{ category.name }}</span></label>

            <label class="list" v-for="item in category.items" :key="item.id" :id="category.id"
                   style="margin-left:9px;display:none;" @change="findLastCategory($event)"
            >
              <input type="checkbox" :id="item.id"><span>
                <router-link :to="item.link" :class="{'sf-menu-item--active': []}">{{ item.name }}</router-link><br></span> </label>
            <div v-for="last in category.last" :key="last.id" ref="children" :id="last.parent_id" style="margin-left:20px;display:none">
              <p :id="category.items">
                <router-link :to="last.link" :class="{'sf-menu-item--active': []}">
                  {{ last.name }}
                </router-link>
                <br>
              </p>
            </div>
            <br>
          </div>
        </SfList>
      </div>
      <div class="products">
        <SfHeading :level="3" :title="$t('Product suggestions')" class="products__title sf-heading--left" />
        <div class="products__listing">
          <SfProductCard
            v-for="product in visibleProducts"
            :key="product.id"
            :title="product.title"
            :image="product.image"
            :regular-price="product.price.regular"
            :special-price="product.price.special"
            :max-rating="product.rating.max"
            :score-rating="product.rating.score"
            :link="product.link"
            :parent="product.productParent"
            link-tag="router-link"
            :wishlist-icon="false"
            class="products__product-card"
            @click.native="$store.commit('ui/setSearchpanel', false)"
          />
        </div>
        <SfButton
          v-if="OnlineOnly && readMore && visibleProducts.length >= pageSize"
          class="sf-button--full-width load-more"
          type="button"
          @click="$emit('see-more')"
        >
          {{ $t("Load more") }}
        </SfButton>
      </div>
    </div>
  </div>
</template>

<script>
import { prepareCategoryMenuItem, getTopLevelCategories } from 'theme/helpers';
import config from 'config';
import i18n from '@vue-storefront/i18n';
import { currentStoreView } from '@vue-storefront/core/lib/multistore';
import { productThumbnailPath } from '@vue-storefront/core/helpers';
import { htmlDecode } from '@vue-storefront/core/filters';
import { formatProductLink } from '@vue-storefront/core/modules/url/helpers';
import { prepareCategoryProduct } from 'theme/helpers';
import VueOfflineMixin from 'vue-offline/mixin';
import { SfHeading, SfButton, SfList, SfMenuItem, SfProductCard } from '@storefront-ui/vue';
import { disableBodyScroll, clearAllBodyScrollLocks } from 'body-scroll-lock';
import { mapGetters } from 'vuex';
import { type } from 'os';
export default {
  name: 'OSearchPanel',
  components: {
    SfButton,
    SfList,
    SfMenuItem,
    SfProductCard,
    SfHeading
  },
  mixins: [VueOfflineMixin],
  data () {
    return {
      selectedCategoryIds: [],
      children: [],
      child: [],
      parents: []
    };
  },
  props: {
    search: {
      type: String,
      required: true,
      default: ''
    },
    pageSize: {
      type: Number,
      required: true,
      default: 16
    },
    products: {
      type: Array,
      required: true,
      default: () => ([])
    },
    readMore: {
      type: Boolean,
      required: true,
      default: false
    }
  },

  computed: {
    visibleProducts () {
      const productList = this.selectedCategoryIds.length
        ? this.products.filter(product => product.category_ids.some(categoryId => this.selectedCategoryIds.includes(categoryId)))
        : this.products;

      return productList.map(product => prepareCategoryProduct(product));
    },
    ...mapGetters({
      getCategories: 'category/getCategories',
      getCategoryProducts: 'category-next/getCategoryProducts'
    }),

    categories () {
      const distinctCategories = this.products
        .filter(product => product.category)
        .map(product => product.category)
        .flat()
        .reduce((result, category) => {
          result[category.category_id] = category;
          return result;
        }, {});

      return Object.values(distinctCategories);
    },
    noResultsMessage () {
      return this.search.length < 3
        ? 'Searched term should consist of at least 3 characters.'
        : !this.visibleProducts.length
          ? 'No results were found.'
          : '';
    },

    parent () {
      return getTopLevelCategories(this.getCategories)
        .map(category => {
          const viewAllMenuItem = {
            ...category,
            name: i18n.t('View all'),
            position: 0
          };
          const subCategories = category.children_data
            ? category.children_data
              .map(subCategory => prepareCategoryMenuItem(
                this.getCategories.find(category => category.id === subCategory.id)
              ))
              .filter(Boolean)
            : [];

          const last = subCategories.map(category => {
            const lastCategory = category.children_data
              ? category.children_data
                .map(subCategory => prepareCategoryMenuItem(
                  this.getCategories.find(category => category.id === subCategory.id)
                ))
                .filter(Boolean)
              : [];
            return lastCategory
          })

          const change = last.flat(1)

          return {
            ...prepareCategoryMenuItem(category),
            items: [prepareCategoryMenuItem(viewAllMenuItem)]
              .concat(subCategories),
            last: change
              .sort((a, b) => a.position - b.position)

          };
        })

        .sort((a, b) => a.position - b.position);
    }
    //   last() {
    //    return  this.children.map(lastCategory=>{
    //     const category = lastCategory.map(last =>{
    //            const thirdCategory = last.children_data
    //             ? last.children_data
    //             .map(sub => prepareCategoryMenuItem(
    //               this.getCategories.find(cate => cate.id === sub.id)
    //               ))
    //               .filter(Boolean)
    //             :[]
    //           return thirdCategory

    //          })
    //           let products = []
    //           let result =[]
    //          for (var i = 0; i < category.length; ++i) {
    //            products  = products.concat(category[i]);
    // }
    //         for (var i = 0; i < products.length; ++i){
    //             if(products[i] !== ''){
    //            result.push(products[i])
    //             }
    //         }

    //           return result.sort((a,b)=> a.parnt_id - b.parent_id)

    //           })

    //  }
  },

  methods: {
    isCategorySelected (category) {
      return this.parent.includes(category.id);
    },
    toggleCategory (category) {
      if (this.isCategorySelected(category)) {
        this.selectedCategoryIds = this.selectedCategoryIds.filter(categoryId => categoryId !== category.id);
      } else {
        this.selectedCategoryIds.push(category.id);
      }
    },
    check (event) {
      const app = document.querySelectorAll('.list')
      console.log(this.parent)
      if (event.target.checked == true) {
        getTopLevelCategories(this.getCategories)
          .map(catagory => {
            if (catagory.id == event.target.id) {
              app.forEach(element => {
                const d = element.getAttribute('id');
                if (d == event.target.id) {
                  element.style.display = 'block'
                } else {
                  element.style.display = 'none'
                }
              });
            }
          })
      } else {
        app.forEach(element => {
          element.style.display = 'none';
        })
      }
    },
    findLastCategory (event) {
      const child = this.$refs.children

      if (event.target.checked == true) {
        child.map(ch => {
          if (ch.id === event.target.id) {
            ch.style.display = 'block'
          } else {
            ch.style.display = 'none'
          }
        })
      } else {
        child.forEach(ch => {
          ch.style.display = 'none'
        })
      }
    },

    isCategoryActive (category) {
      if (!this.getCurrentCategory.path) {
        return false;
      }

      // The 'View all' sub-category (always at position 0) should be marked as active only if it exactly matches current category path,
      // but all other sub-categories will be marked as active when current category path belongs to them.
      return category.position === 0
        ? this.getCurrentCategory.path === category.path
        : this.getCurrentCategory.path.startsWith(category.path);
    }

  },
  watch: {
    parent () {
      this.parents = [];
    },
    categories () {
      this.selectedCategoryIds = [];
    }
  },
  mounted () {
    disableBodyScroll(this.$el)
  },
  destroyed () {
    clearAllBodyScrollLocks()
  }
};
</script>

<style lang="scss" scoped>
@import "~@storefront-ui/shared/styles/helpers/breakpoints";

.o-search-panel {
  position: fixed;
  left: 0;
  right: 0;
  top: var(--_header-height);
  background: var(--c-white);
  overflow: auto;
  max-height: calc(66vh - var(--_header-height));

  @include for-mobile {
    top: auto;
    max-height: calc(100vh - var(--_header-height) - var(--bottom-navigation-height));
  }

  .container {
    display: flex;
    padding: 0 var(--spacer-sm);
    max-width: 1272px;
    margin: auto;
    @include for-desktop {
      border-top: 1px solid var(--c-light);
    }
    @include for-mobile {
      flex-direction: column;
      padding: 0 var(--spacer-xl);
    }
  }

  .categories {
    @include for-desktop {
      flex: 0 0 20%;
      padding-right: 3rem;
      border-right: 1px solid var(--c-light);
    }

    &__title {
      padding: 0;
      margin-top: var(--spacer-base);
      justify-content: start;
    }
    &__listing {
      @include for-desktop {
        margin-top: 2rem;
      }

      .sf-list__item {
        padding: 0.3rem 0;
      }
      .sf-menu-item.selected {
        --menu-item-font-weight: 500;
        text-decoration: underline;
      }
    }
  }

  .products {
    width: 100%;
    &__title {
      padding: 0;
      justify-content: start;
      margin-top: var(--spacer-base);
    }
    &__listing {
      display: flex;
      flex: 0 1 auto;
      flex-wrap: wrap;
    }
    &__product-card {
      --product-card-max-width: 200px;
      flex: 0 1 25%;
      min-width: calc(var(--product-card-max-width) * 0.8);
    }

    @include for-desktop {
      padding-left: 3rem;
    }
  }

  .no-results {
    height: 5rem;
    line-height: 5rem;
    display: flex;
    justify-content: center;
  }

  .load-more {
    margin: var(--spacer-xl) 0;
  }
}
</style>
