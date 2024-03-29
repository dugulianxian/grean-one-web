<script setup lang="ts">
import { useI18n } from "vue-i18n";
import { ref, reactive } from "vue";
import Motion from "../utils/motion";
import { message } from "@/utils/message";
import { updateRules } from "../utils/rule";
import type { FormInstance } from "element-plus";
import { useVerifyCode } from "../utils/verifyCode";
import { $t, transformI18n } from "@/plugins/i18n";
import { useUserStoreHook } from "@/store/modules/user";
import { useRenderIcon } from "@/components/ReIcon/src/hooks";
import Lock from "@iconify-icons/ri/lock-fill";
import Iphone from "@iconify-icons/ep/iphone";

const { t } = useI18n();
const loading = ref(false);
const ruleForm = reactive({
  mobile: "",
  mobileCode: "",
  password: "",
  repeatPassword: ""
});
const ruleFormRef = ref<FormInstance>();
const { isDisabled, text } = useVerifyCode();
const repeatPasswordRule = [
  {
    validator: (rule, value, callback) => {
      if (value === "") {
        callback(new Error(transformI18n($t("login.passwordSureReg"))));
      } else if (ruleForm.password !== value) {
        callback(new Error(transformI18n($t("login.passwordDifferentReg"))));
      } else {
        callback();
      }
    },
    trigger: "blur"
  }
];

const onUpdate = async (formEl: FormInstance | undefined) => {
  loading.value = true;
  if (!formEl) return;
  await formEl.validate((valid, fields) => {
    if (valid) {
      // 模拟请求，需根据实际开发进行修改
      setTimeout(() => {
        message(transformI18n($t("login.passwordUpdateReg")), {
          type: "success"
        });
        loading.value = false;
      }, 2000);
    } else {
      loading.value = false;
      return fields;
    }
  });
};

function onBack() {
  useVerifyCode().end();
  useUserStoreHook().SET_CURRENTPAGE(0);
}
</script>

<template>
  <el-form
    ref="ruleFormRef"
    :model="ruleForm"
    :rules="updateRules"
    size="large"
  >
    <Motion>
      <el-form-item prop="mobile">
        <el-input
          v-model="ruleForm.mobile"
          clearable
          :placeholder="t('login.phone')"
          :prefix-icon="useRenderIcon(Iphone)"
        />
      </el-form-item>
    </Motion>

    <Motion :delay="100">
      <el-form-item prop="mobileCode">
        <div class="w-full flex justify-between">
          <el-input
            v-model="ruleForm.mobileCode"
            clearable
            :placeholder="t('login.smsVerifyCode')"
            :prefix-icon="useRenderIcon('ri:shield-keyhole-line')"
          />
          <el-button
            :disabled="isDisabled"
            class="ml-2"
            @click="useVerifyCode().start(ruleFormRef, 'mobile')"
          >
            {{
              text.length > 0
                ? text + t("login.info")
                : t("login.getVerifyCode")
            }}
          </el-button>
        </div>
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
      <el-form-item :rules="repeatPasswordRule" prop="repeatPassword">
        <el-input
          v-model="ruleForm.repeatPassword"
          clearable
          show-password
          :placeholder="t('login.sure')"
          :prefix-icon="useRenderIcon(Lock)"
        />
      </el-form-item>
    </Motion>

    <Motion :delay="250">
      <el-form-item>
        <el-button
          class="w-full"
          size="large"
          type="primary"
          :loading="loading"
          @click="onUpdate(ruleFormRef)"
        >
          {{ t("login.definite") }}
        </el-button>
      </el-form-item>
    </Motion>

    <Motion :delay="300">
      <el-form-item>
        <el-button class="w-full" size="large" @click="onBack">
          {{ t("login.back") }}
        </el-button>
      </el-form-item>
    </Motion>
  </el-form>
</template>
