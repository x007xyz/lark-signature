<script setup lang="ts">
import vueSignature from "vue3-online-signature";
import {
  bitable,
  FieldType,
  Selection,
  IAttachmentField,
} from "@lark-base-open/js-sdk";
import { dataURItoFile } from "./utils";
import { i18n } from "./locales/i18n";
import { ref, reactive, onMounted } from "vue";

const loading = ref<boolean>(false);

const loadingTip = ref<string>("");

const params = reactive<object>({
  width: 1200,
  height: 500,
  lineWidth: 5,
  lineColor: "",
  canvasBack: "",
  isCrop: false,
  edg: 0,
  fullScreen: false,
  domId: "",
  imgBack: "",
  isRepeat: "",
  noRotation: false,
  backIsCenter: false,
  recoverPoints: [],
});
let vueSignatureRef = ref<any>(null);

let enable = ref<boolean>(false);

let currentSelection: Selection;

let currentField: IAttachmentField;

const handleReset = () => {
  vueSignatureRef.value.reset();
};
const handleGenerate = () => {
  loading.value = true;
  loadingTip.value = i18n.global.t("generate_sign_tip");
  vueSignatureRef.value
    .confirm()
    .then((res: { base64: string; points: any }) => {
      if (currentSelection?.recordId) {
        const file = dataURItoFile(res.base64, "signature.png");
        return currentField!
          .setValue(currentSelection.recordId, file)
          .then(() => {
            // 报错签名后清空画布
            vueSignatureRef.value.reset();
            loading.value = false;
          });
      }
    })
    .catch((err) => {
      loading.value = false;
      // console.log(err); // 画布没有签字时会执行这里 'Not Signned'
    });
};

onMounted(() => {
  bitable.base.onSelectionChange(async ({ data }) => {
    loading.value = true;
    loadingTip.value = i18n.global.t("cell_loading_tip");
    if (data.tableId && data.fieldId) {
      const table = await bitable.base.getTableById(data.tableId);
      const fieldMeta = await table.getFieldMetaById(data.fieldId);
      // 如果是附件类型的字段
      if (fieldMeta.type === FieldType.Attachment) {
        currentSelection = data;
        currentField = await table.getField<IAttachmentField>(data.fieldId);
        enable.value = true;
      } else {
        enable.value = false;
      }
    }
    loading.value = false;
  });
});
</script>

<template>
  <Spin :loading="loading" :tip="loadingTip">
    <div style="position: fixed; width: 100%; top: 0; left: 0">
      <AAlert v-if="!enable" type="warning"> {{ $t("warning_tips") }} </AAlert>
      <AAlert v-else type="success"> {{ $t("success_tips") }} </AAlert>
    </div>
    <div class="signature">
      <vueSignature ref="vueSignatureRef" v-bind="params" />
    </div>
    <div class="btn-wrapper">
      <AButton style="margin-right: 16px" @click="handleReset">
        {{ $t("clear") }}
      </AButton>
      <AButton :disabled="!enable" type="primary" @click="handleGenerate">
        {{ $t("save") }}
      </AButton>
    </div>
  </Spin>
</template>

<style scoped>
.btn-wrapper {
  display: flex;
  justify-content: center;
  margin: 16px 0;
}
.signature {
  border: 1px solid #999;
  border-radius: 4px;
  margin-top: 64px;
}
</style>
