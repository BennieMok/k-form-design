<template>
  <div :style="{ width: record.options.width }" :getFileList="getFileList">
    <a-upload
      :disabled="record.options.disabled"
      v-if="!record.options.drag"
      :name="record.model"
      :multiple="record.options.multiple"
      :data="optionsData"
      :fileList="fileList"
      :action="record.options.action"
      @preview="handlePreview"
      @change="handleChange"
      :remove="remove"
      :beforeUpload="beforeUpload"
    >
      <a-button
        v-if="fileList.length < record.options.limit"
        :disabled="record.options.disabled"
      >
        <a-icon type="upload" /> {{ record.options.placeholder }}
      </a-button>
    </a-upload>
    <a-upload-dragger
      v-else
      :disabled="record.options.disabled"
      :name="record.model"
      :multiple="record.options.multiple"
      :fileList="fileList"
      :data="optionsData"
      :action="record.options.action"
      @preview="handlePreview"
      @change="handleChange"
      :remove="remove"
      :beforeUpload="beforeUpload"
    >
      <p class="ant-upload-drag-icon">
        <a-icon type="cloud-upload" />
      </p>
      <p class="ant-upload-text">单击或拖动文件到此区域</p>
    </a-upload-dragger>
  </div>
</template>
<script>
/*
 * author kcz
 * date 2019-12-31
 * description 上传文件组件
 */
export default {
  // eslint-disable-next-line vue/require-prop-types
  props: ["record", "value"],
  data() {
    return {
      fileList: []
    };
  },
  // watch: {
  //   value(val) {
  //     this.fileList = val;
  //   }
  // },
  computed: {
    getFileList() {
      // 计算value长度，value有值时，修改fileList
      if (this.value) {
        this.setFileList();
        return this.value.length;
      } else {
        return 0;
      }
    },
    optionsData() {
      try {
        return JSON.parse(this.record.options.data);
      } catch {
        return {};
      }
    }
  },
  methods: {
    setFileList() {
      // 当传入value改变时，fileList也要改变
      // 如果传入的值为字符串，则转成json
      if (typeof this.value === "string") {
        this.fileList = JSON.parse(this.value);
        // 将转好的json覆盖组件默认值的字符串
        this.handleSelectChange();
      } else {
        this.fileList = this.value;
      }
    },
    handleSelectChange() {
      setTimeout(() => {
        const arr = this.fileList.map(item => {
          if (typeof item.response !== "undefined") {
            const res = item.response;
            return {
              type: "file",
              name: item.name,
              status: item.status,
              uid: res.data.fileId || new Date().getTime(),
              url: res.data.url || ""
            };
          } else {
            return {
              type: "file",
              name: item.name,
              status: item.status,
              uid: item.uid,
              url: item.url || ""
            };
          }
        });

        this.$emit("change", arr);
        this.$emit("input", arr);
      }, 10);
    },
    handlePreview(file) {
      // 下载文件
      let a = document.createElement("a");
      a.href = file.url || file.thumbUrl;
      a.download = file.name;
      a.click();
    },
    remove() {
      this.handleSelectChange();
    },
    beforeUpload(e, files) {
      if (files.length + this.fileList.length > this.record.options.limit) {
        this.$message.warning(`最大上传数量为${this.record.options.limit}`);
        files.splice(this.record.options.limit - this.fileList.length);
      }
    },
    handleChange(info) {
      this.fileList = info.fileList;
      if (info.file.status === "done") {
        const res = info.file.response;
        if (res.code === 0) {
          this.handleSelectChange();
        } else {
          this.fileList.pop();
          this.$message.error(`文件上传失败`);
        }
      } else if (info.file.status === "error") {
        this.$message.error(`文件上传失败`);
      }
    }
  }
};
</script>
