<template>
  <div id="main-page">
    <el-container>
      <el-header style="height: 30px">
        <el-row :gutter="10">
          <el-col :span="24" style="margin-top: 0">
            <b style="color: red">YouTube</b>
            <b>快捷下载工具</b>
          </el-col>
        </el-row>
      </el-header>
      <el-main>
        <div class="master-area">
          <!-- <SystemInformation></SystemInformation> -->
          <el-row :gutter="20">
            <el-col :span="4">
              <div class="labelName">资源地址</div>
            </el-col>
            <el-col :span="20">
              <div class="grid-content bg-purple">
                <el-input
                  size="mini"
                  v-model="url"
                  placeholder="资源地址"
                  clearable
                ></el-input>
              </div>
            </el-col>
          </el-row>
          <el-row :gutter="20">
            <el-col :span="4">
              <div class="labelName">下载类型</div>
            </el-col>
            <el-col :span="5">
              <div class="grid-content bg-purple">
                <el-select
                  size="mini"
                  v-model="value"
                  clearable
                  placeholder="请选择"
                >
                  <el-option
                    v-for="item in options"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value"
                  ></el-option>
                </el-select>
              </div>
            </el-col>
            <div v-show="value === '音频列表'">
              <el-col :span="3" style="font-size: 12px">
                <div class="labelName">起始序号</div>
              </el-col>
              <el-col :span="4" style="width: 100px">
                <div class="grid-content bg-purple">
                  <el-input size="mini" v-model="startIndex"></el-input>
                </div>
              </el-col>
              <el-col :span="3" style="font-size: 12px">
                <div class="labelName">结束序号</div>
              </el-col>
              <el-col :span="4" style="width: 100px">
                <div class="grid-content bg-purple">
                  <el-input size="mini" v-model="endIndex"></el-input>
                </div>
              </el-col>
            </div>
          </el-row>
          <el-row :gutter="20">
            <el-col :span="4">
              <div class="labelName">下载路径</div>
            </el-col>
            <el-col :span="12">
              <div>
                <el-input
                  size="mini"
                  v-model="output"
                  readonly
                  placeholder="下载路径"
                >
                </el-input>
              </div>
            </el-col>
            <el-col :span="3">
              <div>
                <el-button size="mini" type="primary" @click="selectDWPath"
                  >请选择
                </el-button>
              </div>
            </el-col>
            <el-col :span="4">
              <div>
                <el-button size="mini" type="primary" @click="openDownloadPath"
                  >打开下载目录
                </el-button>
              </div>
            </el-col>
          </el-row>
          <el-row :gutter="20" v-show="btnWord === '正在下载……'">
            <el-col :span="4">
              <div class="labelName">下载进度</div>
            </el-col>
            <el-col :span="20">
              <div class="labelName">
                <el-progress
                  :text-inside="true"
                  :stroke-width="18"
                  :percentage="downloadSchedule.downloadedCompletionRate"
                ></el-progress>
              </div>
            </el-col>
          </el-row>
          <el-row :gutter="20" v-show="btnWord === '正在下载……'">
            <el-col :span="4">
              <div class="labelName"></div>
            </el-col>
            <div style="font-size: 11px">
              <el-col :span="4" style="padding-right: 0">
                <div>
                  正在下载
                  <b>文件{{ downloadSchedule.downloadFileName }}</b>
                </div>
              </el-col>
              <el-col :span="6" style="padding-right: 0">
                <div>
                  文件大小：
                  <b>{{ downloadSchedule.downloadFileSize }}</b>
                </div>
              </el-col>
              <el-col :span="6" style="padding-right: 0">
                <div>
                  下载速度：
                  <b>{{ downloadSchedule.downloadSpeed }}</b>
                </div>
              </el-col>
              <el-col :span="4" style="padding-left: 0; padding-right: 0">
                <div>
                  剩余时间：
                  <b>{{ downloadSchedule.RemainingTime }}</b>
                </div>
              </el-col>
            </div>
          </el-row>
          <el-row :gutter="20">
            <el-col :span="24">
              <div class="download-btn">
                <el-button size="mini" type="success" round @click="runExec">{{
                  btnWord
                }}</el-button>
                <el-button size="mini" type="success" round @click="runStop"
                  >停止下载</el-button
                >
                <!-- <el-button size="mini" type="success" round @click="test"
                  >Test</el-button
                > -->
              </div>
            </el-col>
          </el-row>
        </div>
      </el-main>
    </el-container>
  </div>
</template>

<script>
import SystemInformation from "./LandingPage/SystemInformation";

const spawn = require("child_process").spawn;
const { dialog } = require("electron").remote;
const { shell } = require("electron").remote;
const os = require("os");

export default {
  name: "main-page",
  components: { SystemInformation },
  data() {
    return {
      url: "",
      output: os.homedir() + "\\Desktop\\",
      params: [],
      downloadStatus: "下载完成",
      downloadSchedule: {
        downloadedCompletionRate: 0,
        dcr: 0,
        downloadSpeed: 0,
        RemainingTime: 0,
        downloadFileName: 1,
        downloadFileSize: 0,
        TotalTime: "",
      },
      notify: {
        msg: "",
        type: "success",
      },
      startIndex: 1,
      endIndex: "",
      value: "",
      btnWord: "开始下载",
      stop: false,
      options: [
        {
          value: "",
          label: "",
        },
        {
          value: "音频",
          label: "音频",
        },
        {
          value: "视频",
          label: "视频",
        },
        {
          value: "音频列表",
          label: "音频列表",
        },
      ],
      cmdPath: "extraResources",
      cmdStr: "youtube-dl ",
    };
  },
  methods: {
    open(link) {
      this.$electron.shell.openExternal(link);
    },
    runStop() {
      this.stop = true;
    },
    openDownloadPath() {
      shell.openExternal(this.output);
    },
    selectDWPath() {
      let options = {
        title: "请选择下载路径",
        defaultPath: this.output,
        properties: ["openDirectory", "showHiddenFiles"],
      };
      dialog.showOpenDialog(options, (filePaths) => {
        if (filePaths != undefined) {
          if (filePaths.length > 0) {
            this.output = filePaths[0] + "\\";
          }
        }
      });
    },
    initParam() {
      // 初始化参数
      this.btnWord = "正在下载……";
      this.stop = false;
      this.cmdStr = "youtube-dl";
      this.params = [];
      this.downloadStatus = "下载完成";
      this.downloadSchedule.dcr = 0;
      this.downloadSchedule = {
        downloadedCompletionRate: 0,
        dcr: 0,
        downloadSpeed: 0,
        RemainingTime: 0,
        downloadFileName: 1,
        downloadFileSize: 0,
        TotalTime: "",
      };
    },
    test() {
      console.log(process.cwd());
      if (process.env.NODE_ENV !== "development") {
        this.cmdPath =process.cwd() + "\\extraResources";
      }
      console.log(this.cmdPath);
      // shell.showItemInFolder(this.output);
      // let str = "[download]  98.3% of 18.66MiB at  1.28MiB/s ETA 00:00 ";
      // str = str.match(/of(.*?)at/);
      // console.log(str[1]);
    },
    ProgressProcess(data) {
      let reg = /(\d+\.?\d+)/g;
      let str = data + "";
      let list = str.match(reg);
      console.log(list);
      if (list !== null) {
        if (list.length % 5 === 0) {
          let rate = parseFloat(list[0]);
          // let fileSize = parseFloat(list[1]);
          let fileSize = str.match(/of(.*?)at/)[1];
          // let speed = parseFloat(list[2]);
          let speed = str.match(/at(.*?)ETA/)[1];
          if (rate < 100) {
            this.downloadSchedule.downloadedCompletionRate = rate;
          }
          if (rate == 100) {
            this.downloadSchedule.downloadFileName += 1;
          }
          this.downloadSchedule.downloadFileSize = fileSize;
          this.downloadSchedule.downloadSpeed = speed;
          this.downloadSchedule.RemainingTime = list[3] + ":" + list[4];
        }
      }
    },
    runExec() {
      console.log("已选择" + this.value);
      if (this.url === "") {
        this.$notify({
          title: "请填写资源地址！",
          type: "error",
        });
        return;
      }
      if (this.value === "") {
        this.$notify({
          title: "请选择下载类型",
          type: "error",
        });
        return;
      }

      this.initParam();
      if (this.value === "视频") {
        this.params.push("--ignore-errors");
        this.params.push(this.url);
        this.params.push("-o");
        this.params.push(this.output + "%(title)s.%(ext)s");
      } else if (this.value === "音频") {
        this.params.push("-f");
        this.params.push("140");
        this.params.push("--ignore-errors");
        this.params.push(this.url);
        this.params.push("-o");
        this.params.push(this.output + "%(title)s.%(ext)s");
      } else if (this.value === "音频列表") {
        this.params.push("-f");
        this.params.push("140");
        this.params.push("--ignore-errors");
        this.params.push("--playlist-items");
        this.params.push(this.startIndex + "-" + this.endIndex);
        this.params.push(this.url);
        this.params.push("-o");
        this.params.push(this.output + "%(title)s.%(ext)s");
      }
      let workerProcess;

      if (process.env.NODE_ENV !== "development") {
        this.cmdPath = process.cwd() + "\\resources\\extraResources";
      }
      // 执行命令行，如果命令不需要路径，或就是项目根目录，则不需要cwd参数：
      workerProcess = spawn(this.cmdStr, this.params, {
        cwd: this.cmdPath,
      });

      let that = this;
      workerProcess.stdout.on("data", (data) => {
        console.log("正常信息: " + data);

        this.ProgressProcess(data);
        if (this.stop) {
          that.downloadStatus = "已停止下载";
          process.kill(workerProcess.pid);
        } else if (data.indexOf("has already been downloaded") != -1) {
          that.downloadStatus = "文件已存在";
          that.notify.msg = "请查阅" + that.value;
        }
        that.notify.type = "success";
      });

      // 打印错误的后台可执行程序输出
      workerProcess.stderr.on("data", function (data) {
        console.log("错误信息: " + data);
        // if (that.value !== "音频列表") {
          that.downloadStatus = "下载失败";
          that.notify.msg = "已停止下载！";
          that.notify.type = "error";
        // }
      });

      // 退出之后的输出
      workerProcess.on("close", function (code, signal) {
        // console.log(`下载结束。 code: ${code}; signal: ${signal}`);
        that.$notify({
          title: that.downloadStatus,
          message: that.notify.msg,
          duration: 0,
          type: that.notify.type,
        });
        that.btnWord = "开始下载";
        that.stop = false;
      });
    },
  },
};
</script>

<style>
@import url("https://fonts.googleapis.com/css?family=Source+Sans+Pro");

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}
.download {
  padding-left: 10px;
  padding-right: 0;
}
#main-page {
  width: 100%;
  height: 100%;
}
.el-container {
  width: 100%;
  height: 100%;
}
.el-header {
  background-color: #409eff;
  color: #333;
  text-align: center;
  line-height: 30px;
  padding: 0;
}
.el-main {
  /* background-color: #e9eef3; */
  /* background-color: rgba(49, 148, 230, 0.26); */
  color: #333;
  text-align: left;
  padding: 10px;
  height: 100%;
}
.el-icon-s-home {
  color: #000000;
  cursor: pointer;
}
.master-area {
  width: 600px;
  margin-left: auto;
  margin-right: auto;
}
.el-col {
  margin-top: 6px;
}
.labelName {
  padding: 4px;
}
.download-btn {
  text-align: center;
}
</style>
