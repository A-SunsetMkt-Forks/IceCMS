<script setup lang="ts">
import { useI18n } from "vue-i18n";
import Motion from "./utils/motion";
import { useRouter } from "vue-router";
import { message } from "@/utils/message";
import { loginRules } from "./utils/rule";
import phone from "./components/phone.vue";
import TypeIt from "@/components/ReTypeit";
import { debounce } from "@pureadmin/utils";
import qrCode from "./components/qrCode.vue";
import regist from "./components/regist.vue";
import update from "./components/update.vue";
import { useNav } from "@/layout/hooks/useNav";
import { useEventListener } from "@vueuse/core";
import type { FormInstance } from "element-plus";
import { $t, transformI18n } from "@/plugins/i18n";
import { operates, thirdParty } from "./utils/enums";
import { useLayout } from "@/layout/hooks/useLayout";
import { useUserStoreHook } from "@/store/modules/user";
import { initRouter, getTopMenu } from "@/router/utils";
import { bg, avatar, illustration } from "./utils/static";
import { ReImageVerify } from "@/components/ReImageVerify";
import { ref, toRaw, reactive, watch, computed } from "vue";
import { useRenderIcon } from "@/components/ReIcon/src/hooks";
import { useTranslationLang } from "@/layout/hooks/useTranslationLang";
import { useDataThemeChange } from "@/layout/hooks/useDataThemeChange";

import dayIcon from "@/assets/svg/day.svg?component";
import darkIcon from "@/assets/svg/dark.svg?component";
import globalization from "@/assets/svg/globalization.svg?component";
import Lock from "@iconify-icons/ri/lock-fill";
import Check from "@iconify-icons/ep/check";
import User from "@iconify-icons/ri/user-3-fill";
import Info from "@iconify-icons/ri/information-line";

import * as CommonAPI from "@/api/common/login";

import {
  setTokenFromBackend
} from "@/utils/auth";

defineOptions({
  name: "Login"
});

const imgCode = ref("");
const loginDay = ref<number>(7);
const router = useRouter();
const loading = ref(false);
const checked = ref(false);
const disabled = ref(false);
const ruleFormRef = ref<FormInstance>();
// 判断登录页面显示哪个组件（0：登录（默认）、1：手机登录、2：二维码登录、3：注册、4：忘记密码）
// 使用store中的currentPage，确保组件间可以共享状态
const userStore = useUserStoreHook();
// 初始化时设置为登录页
userStore.SET_CURRENTPAGE(0);
const currentPage = computed(() => userStore.currentPage);

const { t } = useI18n();
const { initStorage } = useLayout();
initStorage();
const { dataTheme, overallStyle, dataThemeChange } = useDataThemeChange();
dataThemeChange(overallStyle.value);
const { title, getDropdownItemStyle, getDropdownItemClass } = useNav();
const { locale, translationCh, translationEn } = useTranslationLang();

const ruleForm = reactive({
  username: "admin",
  password: "admin123",
  verifyCode: ""
});

// 处理登录天数变化
const handleLoginDayChange = (value: number) => {
  console.log("选择的天数:", value);
  loginDay.value = Number(value);
};

const onLogin = async (formEl: FormInstance | undefined) => {
  loading.value = true;
  if (!formEl) return;
  await formEl.validate(async (valid, fields) => {
    if (valid) {
      const response = await CommonAPI.loginByPassword({ username: ruleForm.username, password: ruleForm.password });
      console.log(response);
       if (response.code === 200) {
          // 更新store中的记住登录状态
          userStore.SET_ISREMEMBERED(checked.value);
          userStore.SET_LOGINDAY(Number(loginDay.value));

          console.log("记住登录:", checked.value, "天数:", Number(loginDay.value));

          // 登录成功后 将token存储到sessionStorage中
          setTokenFromBackend(response.data, checked.value, Number(loginDay.value));

          // 获取后端路由
          initRouter().then(() => {
            router.push(getTopMenu(true).path);
            message("登录成功", { type: "success" });
            // 登录成功后刷新页面
            window.location.reload();
          });

        }else{
           //如果登录失败则重新获取验证码
          // getCaptchaCode();
          loading.value = false;
          // getTopMenu(true).path;
          message("登录失败", { type: "error" });

        }


      //   CommonAPI.loginByPassword({
      //     username: ruleForm.username,
      //     password: ruleForm.password
      //   })
      //   .then(({ data }) => {
      //     console.log(data);
      //      // 获取后端路由
      //     initRouter().then(() => {
      //       router.push(getTopMenu(true).path);
      //       message("登录成功", { type: "success" });
      //       // 登录成功后刷新页面
      //       window.location.reload();
      //     });
      //     // 登录成功后 将token存储到sessionStorage中
      //     setTokenFromBackend(data);
      //   }
      // )

        // .catch(() => {
        //   loading.value = false;
        //   //如果登录失败则重新获取验证码
        //   // getCaptchaCode();
        // });

      // useUserStoreHook()
        // .loginByUsername({ username: ruleForm.username, password: "admin123" })
        // .then(res => {
        //   if (res.success) {
        //     // 获取后端路由
        //     return initRouter().then(() => {
        //       disabled.value = true;
        //       router
        //         .push(getTopMenu(true).path)
        //         .then(() => {
        //           message("登录成功", { type: "success" });
        //         })
        //         .finally(() => (disabled.value = false));
        //     });
        //   }
        // })
        // .finally(() => (loading.value = false));
    } else {
      loading.value = false;
      return fields;
    }
  });
};

const immediateDebounce: any = debounce(
  formRef => onLogin(formRef),
  1000,
  true
);

useEventListener(document, "keypress", ({ code }) => {
  if (code === "Enter" && !disabled.value && !loading.value)
    immediateDebounce(ruleFormRef.value);
});

watch(imgCode, value => {
  useUserStoreHook().SET_VERIFYCODE(value);
});
watch(checked, bool => {
  useUserStoreHook().SET_ISREMEMBERED(bool);
});
watch(loginDay, value => {
  useUserStoreHook().SET_LOGINDAY(value);
});
</script>

<template>
  <div class="select-none modern-login" :class="{ 'dark': dataTheme }">
    <img :src="bg" class="wave" />
    <div class="top-toolbar">
      <!-- 主题 -->
      <el-switch
        v-model="dataTheme"
        inline-prompt
        :active-icon="dayIcon"
        :inactive-icon="darkIcon"
        @change="dataThemeChange"
      />
      <!-- 国际化 -->
      <el-dropdown trigger="click">
        <globalization
          class="hover:text-primary hover:!bg-[transparent] w-[20px] h-[20px] cursor-pointer outline-none duration-300"
        />
        <template #dropdown>
          <el-dropdown-menu class="translation">
            <el-dropdown-item
              :style="getDropdownItemStyle(locale, 'zh')"
              :class="['dark:!text-white', getDropdownItemClass(locale, 'zh')]"
              @click="translationCh"
            >
              <IconifyIconOffline
                v-show="locale === 'zh'"
                class="check-zh"
                :icon="Check"
              />
              简体中文
            </el-dropdown-item>
            <el-dropdown-item
              :style="getDropdownItemStyle(locale, 'en')"
              :class="['dark:!text-white', getDropdownItemClass(locale, 'en')]"
              @click="translationEn"
            >
              <span v-show="locale === 'en'" class="check-en">
                <IconifyIconOffline :icon="Check" />
              </span>
              English
            </el-dropdown-item>
          </el-dropdown-menu>
        </template>
      </el-dropdown>
    </div>
    <div class="login-container">
      <div class="img">
        <component :is="toRaw(illustration)" />
      </div>
      <div class="login-box">
        <div class="login-form" :class="{'login-form-en': locale === 'en'}">
          <!-- <avatar class="avatar" /> -->
          <Motion>
            <h2 class="outline-none">
              <TypeIt :values="[title]" :cursor="false" :speed="150" />
            </h2>
          </Motion>

          <el-form
            v-if="currentPage === 0"
            ref="ruleFormRef"
            :model="ruleForm"
            :rules="loginRules"
            size="large"
          >
            <Motion :delay="100">
              <el-form-item
                :rules="[
                  {
                    required: true,
                    message: transformI18n($t('login.usernameReg')),
                    trigger: 'blur'
                  }
                ]"
                prop="username"
              >
                <el-input
                  v-model="ruleForm.username"
                  clearable
                  :placeholder="t('login.username')"
                  :prefix-icon="useRenderIcon(User)"
                />
              </el-form-item>
            </Motion>

            <Motion :delay="150">
              <el-form-item prop="password">
                <el-input
                  v-model="ruleForm.password"
                  clearable
                  show-password
                  :placeholder="t('login.password')"
                  :prefix-icon="useRenderIcon(Lock)"
                />
              </el-form-item>
            </Motion>

            <Motion :delay="200">
              <el-form-item prop="verifyCode">
                <el-input
                  v-model="ruleForm.verifyCode"
                  clearable
                  :placeholder="t('login.verifyCode')"
                  :prefix-icon="useRenderIcon('ri:shield-keyhole-line')"
                >
                  <template v-slot:append>
                    <ReImageVerify v-model:code="imgCode" />
                  </template>
                </el-input>
              </el-form-item>
            </Motion>

            <Motion :delay="250">
              <el-form-item>
                <div class="w-full flex justify-between items-center">
                  <el-checkbox v-model="checked">
                    <div class="flex items-center flex-wrap" :class="{'remember-login-en': locale === 'en'}">
                      <el-select
                        v-model="loginDay"
                        size="small"
                        class="remember-select mr-1"
                        :class="{'w-16': locale === 'zh', 'w-20': locale === 'en'}"
                        :disabled="!checked"
                        @change="handleLoginDayChange"
                      >
                        <el-option :value="1" :label="'1'" />
                        <el-option :value="7" :label="'7'" />
                        <el-option :value="30" :label="'30'" />
                      </el-select>
                      <span class="text-sm remember-text">{{ t("login.remember") }}</span>
                      <el-tooltip
                        effect="dark"
                        placement="top"
                        :content="t('login.rememberInfo')"
                      >
                        <IconifyIconOffline :icon="Info" class="ml-1 text-gray-400" />
                      </el-tooltip>
                    </div>
                  </el-checkbox>
                  <el-button
                    link
                    type="primary"
                    class="text-sm font-normal forget-btn"
                    :class="{'text-xs': locale === 'en'}"
                    @click="userStore.SET_CURRENTPAGE(4)">
                    {{ t("login.forget") }}
                  </el-button>
                </div>
                <el-button
                  class="w-full mt-4"
                  size="large"
                  type="primary"
                  :loading="loading"
                  :disabled="disabled"
                  @click="onLogin(ruleFormRef)"
                >
                  {{ t("login.login") }}
                </el-button>
              </el-form-item>
            </Motion>

            <Motion :delay="300">
              <el-form-item>
                <div class="w-full flex justify-between items-center gap-2 mt-4">
                  <el-button
                    v-for="(item, index) in operates"
                    :key="index"
                    class="flex-1 operation-btn"
                    :class="{'text-xs': locale === 'en'}"
                    size="default"
                    text
                    @click="userStore.SET_CURRENTPAGE(item.page)"
                  >
                    {{ t(item.title) }}
                  </el-button>
                </div>
              </el-form-item>
            </Motion>
          </el-form>

          <Motion v-if="currentPage === 0" :delay="350">
            <el-form-item>
              <el-divider>
                <p class="text-gray-500 text-xs">{{ t("login.thirdLogin") }}</p>
              </el-divider>
              <div class="third-party-icons">
                <div
                  v-for="(item, index) in thirdParty"
                  :key="index"
                  class="icon-wrapper"
                  :title="t(item.title)"
                >
                  <IconifyIconOnline
                    :icon="`ri:${item.icon}-fill`"
                    width="20"
                    class="text-gray-600 hover:text-blue-500"
                  />
                </div>
              </div>
            </el-form-item>
          </Motion>
          <!-- 手机号登录 -->
          <phone v-if="currentPage === 1" />
          <!-- 二维码登录 -->
          <qrCode v-if="currentPage === 2" />
          <!-- 注册 -->
          <regist v-if="currentPage === 3" />
          <!-- 忘记密码 -->
          <update v-if="currentPage === 4" />
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
@import url("@/style/modern-login.scss");

:deep(.el-input-group__append, .el-input-group__prepend) {
  padding: 0;
}

.translation {
  ::v-deep(.el-dropdown-menu__item) {
    padding: 5px 40px;
  }

  .check-zh {
    position: absolute;
    left: 20px;
  }

  .check-en {
    position: absolute;
    left: 20px;
  }
}
</style>
