<template>
  <div class="navbar">
    <hamburger id="hamburger-container" :is-active="sidebar.opened" class="hamburger-container"
               @toggleClick="toggleSideBar"/>

    <breadcrumb id="breadcrumb-container" class="breadcrumb-container"/>

    <div class="right-menu">
      <template v-if="device!=='mobile'">
        <!--        <search id="header-search" class="right-menu-item" />-->

        <!-- <error-log class="errLog-container right-menu-item hover-effect"/> -->

        <!--        <screenfull id="screenfull" class="right-menu-item hover-effect" />-->

        <!--        <el-tooltip content="Global Size" effect="dark" placement="bottom">-->
        <!--          <size-select id="size-select" class="right-menu-item hover-effect" />-->
        <!--        </el-tooltip>-->
        
      </template>
      <el-select class="select-tenant" v-model="tenant.resource" filterable @change="changeTenant">
        <el-option
          v-for="(item, index) in tenantList"
          :key="index"
          :label="item.resource"
          :value="index">
        </el-option>
      </el-select>
      <langChange />
      <el-dropdown class="avatar-container right-menu-item hover-effect" trigger="click">
        <div class="avatar-wrapper">
          <img src="../../../public/hippo4j.gif" class="user-avatar">
          <i class="el-icon-caret-bottom"/>
        </div>
        <el-dropdown-menu slot="dropdown">
          <!--<router-link to="/profile/index">
            <el-dropdown-item>Profile</el-dropdown-item>
          </router-link>-->
          <router-link to="/">
            <el-dropdown-item>{{ this.$t('menu.dashboard') }}</el-dropdown-item>
          </router-link>
          <el-dropdown-item divided>
            <span style="display:block;" @click="logout">{{ $t('system.logOut') }}</span>
          </el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
    </div>
  </div>
</template>

<script>
  import * as jobProjectApi from '@/api/hippo4j-tenant';
  import * as user from '@/api/hippo4j-user';
  import { mapGetters, mapState } from 'vuex'
  import Breadcrumb from '@/components/Breadcrumb'
  import Hamburger from '@/components/Hamburger'
  import ErrorLog from '@/components/ErrorLog'
  import langChange from '@/locale/langChange'
  import { i18nConfig } from '@/locale/config'

  export default {
    data() {
      return {
        tenant: {
          action: '',
          resource: '',
          username: ''
        }
      }
    },
    components: {
      Breadcrumb,
      Hamburger,
      ErrorLog,
      langChange
    },
    computed: {
      ...mapGetters([
        'sidebar',
        'avatar',
        'device',
        'tenantList',
        'tenantInfo'
      ])
    },
    watch: {
      tenantInfo(newVal) {
        this.tenant.tenantId = newVal.tenantId
        this.tenant.resource = newVal.resource
        console.log("ischangLang", newVal)
      }
    },
    methods: {
      toggleSideBar() {
        this.$store.dispatch('app/toggleSideBar')
      },
      async logout() {
        this.$cookie.delete('userName')
        await this.$store.dispatch('user/logout')
        this.$router.push(`/login?redirect=${this.$route.fullPath}`)
      },
      async getTenantList() {
        const userName = this.$cookie.get('userName')
        await user
        .getCurrentUser(userName)
        .then((response) => {
          const { resources } = response;
          resources.map((item) => ({
            ...item,
            tenantId: item.resource
          }))
          if (response.role == 'ROLE_ADMIN') {
            resources.unshift({
              action: "rw",
              resource: this.$t('common.allTenant'),
              username: userName,
              tenantId: this.$t('common.allTenant'),
            })
          }
          this.$store.dispatch('tenant/setTenantList', resources)
          this.$store.dispatch('tenant/setTenantInfo', this.tenantInfo || resources[0])
          this.tenant = this.tenantInfo || resources[0]
          console.log("isResour", resources[0], this.tenant)
        })
        .catch(() => {});
        
      },
      async changeTenant(index) {
        console.log("isTenList", this.tenantList)
        let tenant = {
          tenantId: this.tenantList[index].resource,
          resource: this.tenantList[index].resource,
          current: 1,
          desc: true,
          size: 10,
        }
        this.$store.dispatch('tenant/setTenantInfo', tenant)
        let isAllTenant = tenant.tenantId == i18nConfig.messages.zh.common.allTenant || tenant.tenantId == i18nConfig.messages.en.common.allTenant
        tenant.tenantId = isAllTenant ? '' : tenant.tenantId
        await jobProjectApi.list(tenant).then((response) => {
          console.log("isRes", response)
          // this.$store.dispatch('tenant/setTenantList', resources)
        });
      }
    },
    async mounted() {
      this.getTenantList()
    },
  }
</script>

<style lang="scss" scoped>
  .navbar {
    height: 50px;
    overflow: hidden;
    position: relative;
    background: #fff;
    box-shadow: 0 1px 4px rgba(0, 21, 41, 0.08);

    .hamburger-container {
      line-height: 46px;
      height: 100%;
      float: left;
      cursor: pointer;
      transition: background 0.3s;
      -webkit-tap-highlight-color: transparent;

      &:hover {
        background: rgba(0, 0, 0, 0.025);
      }
    }

    .breadcrumb-container {
      float: left;
    }

    .errLog-container {
      display: inline-block;
      vertical-align: top;
    }

    .right-menu {
      float: right;
      height: 100%;
      line-height: 50px;
      display: flex;
      &:focus {
        outline: none;
      }

      ::v-deep  .el-input__inner {
        border: none;
        box-shadow: none;
      }

      .select-tenant {
        margin-right: 20px;
        border: 0;
        width: 150px;
      }

      .right-menu-item {
        display: inline-block;
        padding: 0 8px;
        height: 100%;
        font-size: 18px;
        color: #5a5e66;
        vertical-align: text-bottom;

        &.hover-effect {
          cursor: pointer;
          transition: background 0.3s;

          &:hover {
            background: rgba(0, 0, 0, 0.025);
          }
        }
      }

      .avatar-container {
        margin-right: 30px;

        .avatar-wrapper {
          margin-top: 5px;
          position: relative;

          .user-avatar {
            cursor: pointer;
            width: 40px;
            height: 40px;
            border-radius: 10px;
          }

          .el-icon-caret-bottom {
            cursor: pointer;
            position: absolute;
            right: -20px;
            top: 25px;
            font-size: 12px;
          }
        }
      }
    }
  }
</style>
