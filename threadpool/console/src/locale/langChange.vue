<template>
  <div class="lang-drop-wrap">
    <el-dropdown @command="selectedLang" trigger="click">
      <span class="el-dropdown-link">
        {{ currentLangName }}
        <i class="el-icon-arrow-down el-icon--right"></i>
      </span>
      <el-dropdown-menu slot="dropdown">
        <el-dropdown-item
          v-for="item in langSelectList"
          :key="item.lang"
          :command="item.lang"
        >
          <span class="dropdown-item-text" :data-active="item.lang === currentLang">{{ item.name }}</span>
        </el-dropdown-item>
      </el-dropdown-menu>
    </el-dropdown>
  </div>
</template>
<script>
import { langSelectList, i18nConfig } from './config'
import * as user from '@/api/hippo4j-user';
import { mapGetters } from 'vuex';
export default {
  data() {
    return {
      langSelectList: langSelectList(),
      currentLang: ''
    }
  },
  mounted() {
    const lang = this.$i18n.locale
    this.currentLang = lang || null
  },
  computed: {
    ...mapGetters(['tenantInfo', 'tenantList']),
    currentLangName() {
      const langItem = this.langSelectList.find(item => item.lang === this.currentLang)
      return langItem?.name || '选择语言'
    }
  },
  methods: {
    // 选择语言
    selectedLang(value) {
      this.currentLang = value
      this.$i18n.locale = value
      localStorage.setItem('locale_lang', value)
      this.changeTenant()
    },

    changeTenant() {
      const userName = this.$cookie.get('userName');
      user.getCurrentUser(userName)
      .then((response) => {
        const { resources } = response;
        resources.map((item) => ({
          ...item,
          tenantId: item.resource
        }))

        //change lang =》 change global tenantInfo
        if (response.role == 'ROLE_ADMIN') {
          //判断tenantInfo是否为所有租户选项
          let tenantId = this.tenantInfo.resource
          let isAllTenant = tenantId == i18nConfig.messages.zh.common.allTenant || tenantId == i18nConfig.messages.en.common.allTenant
          let alreadyHasAll = resources[0] == i18nConfig.messages.zh.common.allTenant || resources[0] == i18nConfig.messages.en.common.allTenant        
          if (alreadyHasAll) {
            this.$set(resources[0], 'resource', this.$t('common.allTenant'))
            this.$store.dispatch('tenant/setTenantList', resources)
          } else {
            resources.unshift({
              action: "rw",
              resource: this.$t('common.allTenant'),
              username: userName,
              tenantId: this.$t('common.allTenant'),
            })
            this.$store.dispatch('tenant/setTenantList', resources)
          }
          if (isAllTenant) {
            this.$store.dispatch('tenant/setTenantInfo', resources[0])
          }
        }
      })
    }
  }
}
</script>
<style lang="scss" scoped>
  .lang-drop-wrap{
    height: 100%;
    margin-right: 15px;
    .el-dropdown-link{
      cursor: pointer;
    }
  }
  .dropdown-item-text{
    &[data-active=true] {
      color: var(--jjext-color-dropdown-text) !important;
    }
  }
</style>
