<!--
 * @Description:
 * @Version: 2.0
 * @Author: yichuanhao
 * @Date: 2022-06-13 17:29:58
 * @LastEditors: yichuanhao
 * @LastEditTime: 2022-07-26 15:14:22
-->
<template>
  <div class="common__dialog-wrap">
    <el-dialog
      :visible.sync="showDialog"
      :close-on-click-modal="false"
      :width="dialogWidth"
      :title="dialogTitle"
      custom-class="common__dialog"
      @close="dialogClose"
      :modal="modal"
    >
      <!-- 内容插槽区域 -->
      <slot></slot>
      <div slot="footer" class="dialog-footer" v-if="isShowFooterBtn">
        <!-- 取消按钮 -->
        <el-button size="small" @click="dialogClose" style="border: none" v-if="isShowCancleBtn">{{ $t('common.cancel') }}</el-button>
        <!-- 其他按钮插槽 -->
        <slot name="footerbtn"></slot>
        <!-- 确定按钮 -->
        <el-button size="small" type="primary" @click="dialogConfirm" v-if="isShowConfirmBtn">{{ $t('common.ok') }}</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
/**
 * @dialogTitle String 弹窗标题
 * @dialogWidth String 弹窗宽度
 * @isShowCancleBtn Boolean 是否展示取消按钮
 * @isShowConfirmBtn Boolean 是否展示确定按钮
 * @isShowFooterBtn Boolean 是否展示底部按钮
 * @modal Boolean 是否展示遮罩层
 */
export default {
  // 公共弹窗
  name: 'commonDialog',
  props: {
    dialogTitle: {
      type: String,
      default: '',
    },
    dialogWidth: {
      type: String,
      default: '400px',
    },
    isShowCancleBtn: {
      type: Boolean,
      default: true,
    },
    isShowConfirmBtn: {
      type: Boolean,
      default: true,
    },
    isShowFooterBtn: {
      type: Boolean,
      default: true,
    },
    modal: {
      type: Boolean,
      default: true,
    },
  },
  data() {
    return {
      showDialog: false, // 通过refs获取该实例进行修改，例如：this.$refs.refTitle.showDialog = false;
    };
  },
  methods: {
    dialogClose() {
      this.$emit('dialogClose');
    },
    dialogConfirm() {
      this.$emit('dialogConfirm');
    },
  },
};
</script>

<style>
.el-dialog__wrapper {
  z-index: 100001 !important;
}
</style>
