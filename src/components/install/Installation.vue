<template>
  <div id="installation" class="body-wrapper">
    <el-row type="flex" class="row-bg" justify="center">
      <el-col v-loading="installationLoading">
        <h1>Cluster Installation</h1>
        <div class="form-wrapper">
          <div class="form">
            <el-form
              :model="installationParam"
              :rules="rules"
              ref="installationParam"
              label-width="150px"
              class="demo-ruleForm"
              size="medium"
            >
              <el-form-item label="Cluster Name" prop="clusterName">
                <el-input v-model="installationParam.clusterName" show-word-limit></el-input>
              </el-form-item>
              <el-form-item label="Redis Password" prop="redisPassword">
                <el-input v-model.trim="installationParam.redisPassword" show-word-limit></el-input>
              </el-form-item>
              <!-- <el-form-item label="Redis Mode" prop="redisMode">
                <el-radio-group v-model="installationParam.redisMode">
                  <el-radio label="cluster">Cluster</el-radio>
                  <el-radio label="standalone">Standalone</el-radio>
                </el-radio-group>
              </el-form-item>-->
              <!-- environment start -->
              <!-- <el-form-item label="Environment" prop="installationEnvironment">
                <el-radio-group v-model="installationParam.installationEnvironment">
                  <el-radio
                    v-for="environment in installationEnvironmentList"
                    :key="environment.type"
                    :label="environment.type"
                  >{{ environment.name }}</el-radio>
                </el-radio-group>
              </el-form-item>-->
              <!-- environment end -->
              <!-- image start -->
              <el-form-item
                label="Image"
                prop="image"
                v-if="installationParam.installationEnvironment == 0"
              >
                <el-select
                  allow-create
                  filterable
                  v-model.trim="installationParam.image"
                  placeholder="Please choose image"
                >
                  <el-option
                    v-for="image in dockerImages"
                    :key="image"
                    :label="image"
                    :value="image"
                  ></el-option>
                </el-select>
              </el-form-item>
              <el-form-item
                label="Image"
                prop="image"
                v-else-if="installationParam.installationEnvironment == 1"
              >
                <el-select v-model.trim="installationParam.image" placeholder="Please choose image">
                  <el-option
                    v-for="image in machineImages"
                    :key="image"
                    :label="image"
                    :value="image"
                  ></el-option>
                </el-select>
              </el-form-item>
              <el-form-item
                label="Image"
                prop="image"
                v-if="installationParam.installationEnvironment == 3"
              >
                <el-select
                  allow-create
                  filterable
                  v-model.trim="installationParam.image"
                  placeholder="Please choose image"
                >
                  <el-option
                    v-for="image in humpbackImages"
                    :key="image"
                    :label="image"
                    :value="image"
                  ></el-option>
                </el-select>
              </el-form-item>
              <!-- image end -->

              <!-- <el-form-item label="Auto Build" prop="autoBuild">
                <el-switch v-model="installationParam.autoBuild"></el-switch>
              </el-form-item>-->
              <!-- auto install start -->

              <el-form-item label="Upload redis.conf">
                <el-upload
                  class="upload-demo"
                  drag
                  action="https://jsonplaceholder.typicode.com/posts/"
                  multiple
                >
                  <i class="el-icon-upload"></i>
                  <div class="el-upload__text">
                    <em>click to upload</em>
                  </div>
                </el-upload>
              </el-form-item>

              <el-form-item label="Machine List" prop="machines" v-if="installationParam.autoBuild">
                <div class="block">
                  <el-cascader
                    placeholder="Search"
                    :options="allMachineList"
                    :props="{ multiple: true }"
                    filterable
                    :leafOnly="true"
                    @change="handleMachine"
                    v-model="installationParam.machines"
                  ></el-cascader>
                </div>
              </el-form-item>

              <el-form-item label="Master Port" prop="startPort" v-if="installationParam.autoBuild">
                <el-input
                  v-model="installationParam.startPort"
                  placeholder="e.g. 10001,10002,10003"
                ></el-input>
              </el-form-item>
              <el-form-item label="Slave Port">
                <el-input
                  v-model="installationParam.replicaNumber"
                  placeholder="e.g. 10004,10005,10006"
                ></el-input>
              </el-form-item>

              <!--<el-form-item-->
              <!--label="Master Number"-->
              <!--prop="masterNumber"-->
              <!--v-if="installationParam.autoBuild"-->
              <!--&gt;-->
              <!--<el-input v-model.number="installationParam.masterNumber"></el-input>-->
              <!--</el-form-item>-->
              <!--<el-form-item-->
              <!--label="Replica Number"-->
              <!--prop="replicaNumber"-->
              <!--v-if="installationParam.autoBuild"-->
              <!--&gt;-->
              <!--<el-input v-model.number="installationParam.replicaNumber"></el-input>-->
              <!--</el-form-item>-->

              <!-- auto install end -->
              <!-- custom installation start -->
              <el-form-item
                label="Machine User"
                prop="machineUserName"
                v-if="!installationParam.autoBuild"
              >
                <el-input
                  v-model.trim="installationParam.machineUserName"
                  placeholder="Machine user name"
                ></el-input>
              </el-form-item>
              <el-form-item
                label="Machine Password"
                prop="machinePassword"
                v-if="!installationParam.autoBuild"
              >
                <el-input
                  v-model.trim="installationParam.machinePassword"
                  placeholder="Machine password"
                ></el-input>
              </el-form-item>
              <el-form-item label="Topology" prop="redisNodes" v-if="!installationParam.autoBuild">
                <el-input
                  type="textarea"
                  :autosize="{ minRows: 2, maxRows: 500}"
                  placeholder="Please enter redis node list"
                  v-model="installationParam.redisNodes"
                  class="textarea"
                ></el-input>
              </el-form-item>

              <!-- custom installation end -->
              <!-- <el-form-item label="Sudo" prop="sudo">
                <el-switch v-model="installationParam.sudo"></el-switch>
                <el-tooltip
                  class="item"
                  effect="dark"
                  content="Whether to use sudo during the installation operation"
                  placement="top-start"
                >
                  <i class="el-icon-info info"></i>
                </el-tooltip>
              </el-form-item>-->
              <!-- <el-form-item
                label="Init Slot"
                prop="autoInit"
                v-if="installationParam.redisMode == 'cluster'"
              >
                <el-switch v-model="installationParam.autoInit"></el-switch>
                <el-tooltip
                  class="item"
                  effect="dark"
                  content="Automatic allocation slot"
                  placement="top-start"
                >
                  <i class="el-icon-info info"></i>
                </el-tooltip>
              </el-form-item>-->
              <el-form-item>
                <el-button type="success" @click="installationCheck('installationParam')">Install</el-button>
                <!-- <el-button @click="resetForm('installationParam')">Reset</el-button> -->
              </el-form-item>
            </el-form>
          </div>
        </div>
      </el-col>
      <!-- <el-col :xl="12" :lg="12" :md="24" :sm="24">
        <div class="console-wrapper">
          <div class="console-title">Redis Installation Console</div>
          <pre class="console">{{ installationConsole }}</pre>
        </div>
      </el-col>-->
    </el-row>
    <!-- <el-dialog title="Installation Params" :visible.sync="installationInfoVisible" width="50%">
      <div class="item-param">
        <span class="param-key">Group:</span>
        <el-tag size="mini">{{ currentGroup.groupName }}</el-tag>
      </div>
      <div class="item-param">
        <span class="param-key">Cluster Name:</span>
        <el-tag size="mini">{{ installationParam.clusterName }}</el-tag>


      </div>
      <div class="item-param">
        <span class="param-key">Redis Mode:</span>
        <el-tag size="mini">{{ installationParam.redisMode }}</el-tag>
      </div>
      <div v-if="installationParam.autoBuild">
        <div class="item-param">
          <span class="param-key">Master Number:</span>
          <el-tag size="mini">{{ installationParam.masterNumber }}</el-tag>
        </div>
        <div class="item-param">
          <span class="param-key">Replica Number:</span>
          <el-tag size="mini">{{ installationParam.replicaNumber }}</el-tag>
        </div>
      </div>
      <div v-if="installationParam.installationEnvironment == 0" class="item-param">
        <span class="param-key">Environment:</span>
        <el-tag size="mini">Docker</el-tag>
      </div>
      <div v-else-if="installationParam.installationEnvironment == 1" class="item-param">
        <span class="param-key">Environment:</span>
        <el-tag size="mini">Machine</el-tag>
      </div>
      <div class="item-param">
        <span class="param-key">Image:</span>
        {{ installationParam.image }}
      </div>
      <div class="item-param">
        <div class="param-key">Topology:</div>
        {{ installationParam.redisNodes }}
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button size="small" @click="installationInfoVisible = false">Cancel</el-button>
        <el-button size="small" type="primary" @click="installationInfoVisible = false">Install</el-button>
      </span>
    </el-dialog>-->
  </div>
</template>

<script>
import { store } from "@/vuex/store.js";
import { isEmpty, validateIpAndPort, validatePort } from "@/utils/validate.js";
import API from "@/api/api.js";
import apiConfig from "@/api/apiConfig.js";
import axios from "axios";
import message from "@/utils/message.js";
export default {
  data() {
    var validateClusterName = (rule, value, callback) => {
      if (isEmpty(value) || isEmpty(value.trim())) {
        return callback(new Error("Please enter cluster name"));
      } else {
        let url = "/cluster/validateClusterName/" + value;
        API.get(
          url,
          null,
          response => {
            let result = response.data;
            if (result.code != 0) {
              let cluster = result.data;
              return callback(new Error(value + " has exist"));
            } else {
              callback();
            }
          },
          err => {
            return callback(new Error("Network error, " + err));
          }
        );
      }
    };
    var validateClusterNameCurrentUsed = (rule, value, callback) => {
      if (isEmpty(value) || isEmpty(value.trim())) {
        return callback(new Error("Please enter cluster name"));
      } else {
        let url = "/installation/validateClusterName/" + value;
        API.get(
          url,
          null,
          response => {
            let result = response.data;
            if (result.code != 0) {
              let cluster = result.data;
              return callback(new Error(value + " is being installed"));
            } else {
              callback();
            }
          },
          err => {
            return callback(new Error("Network error, " + err));
          }
        );
      }
    };
    var validateStartPort = (rule, value, callback) => {
      if (true) {
        return callback(new Error("Incorrect port format"));
      }
      callback();
    };
    var validateMasterAndReplicaNumber = (rule, value, callback) => {
      if (value <= 0) {
        return callback(new Error("Number must be greater than 0"));
      }
      callback();
    };
    var validateMasterNumber = (rule, value, callback) => {
      if (this.installationParam.redisMode == "standalone" && value > 1) {
        return callback(new Error("Standalone mode only need 1 master"));
      }
      callback();
    };
    var validateTopology = (rule, value, callback) => {
      let topology = value.trim();
      if (isEmpty(topology)) {
        return callback(new Error("Please enter topology"));
      }
      let machineUserName = this.installationParam.machineUserName;
      let machinePassword = this.installationParam.machinePassword;
      if (isEmpty(machineUserName) || isEmpty(machinePassword)) {
        return callback(
          new Error("Please enter machine user name and password first")
        );
      }
      let nodeListWithRole = topology.split(/[(\r\n)\r\n]+/);
      let size = nodeListWithRole.length;
      let machineList = [];
      let redisNodeList = [];
      for (var i = 0; i < size; i++) {
        let nodeAndRole = nodeListWithRole[i].split(/\s+/);
        let oneLineSize = nodeAndRole.length;

        if (i == 0) {
          if (oneLineSize != 2) {
            return callback(new Error("Line " + (i + 1) + ": wrong format"));
          } else if (nodeAndRole[1] != "master") {
            return callback(new Error("Line " + (i + 1) + ": node not master"));
          }
        }
        if (
          i != 0 &&
          this.installationParam.redisMode == "standalone" &&
          nodeAndRole[1] == "master"
        ) {
          return callback(
            new Error(
              "Line " + (i + 1) + ": standalone mode only need one master"
            )
          );
        }
        let nodeRole = nodeAndRole[1];
        if (isEmpty(nodeRole)) {
          nodeRole = "slave";
        }
        let ipAndPort = nodeAndRole[0].split(":");
        if (validateIpAndPort(ipAndPort)) {
          return callback(new Error("Line " + (i + 1) + ": wrong format"));
        }
        let host = ipAndPort[0];
        let port = ipAndPort[1];
        let machine = {
          userName: machineUserName,
          password: machinePassword,
          host: host
        };
        let inMachineList = false;
        machineList.forEach(machine => {
          if (machine.host == host) {
            inMachineList = true;
            return;
          }
        });
        if (!inMachineList) {
          machineList.push(machine);
        }
        let redisNodeRepeat = false;
        redisNodeList.forEach(redisNode => {
          if (redisNode.host == host && redisNode.port == port) {
            redisNodeRepeat = true;
          }
        });
        if (redisNodeRepeat) {
          return callback(new Error("Line " + (i + 1) + ": redis node repeat"));
        } else {
          redisNodeList.push({
            host: host,
            port: port,
            nodeRole: nodeRole.toLocaleUpperCase()
          });
        }
      }
      this.installationParam.machineList = machineList;
      this.installationParam.redisNodeList = redisNodeList;
      callback();
    };
    return {
      endPort: "",
      dockerImages: [],
      machineImages: [],
      humpbackImages: [],
      installationParam: {
        groupId: "",
        clusterName: "",
        redisPassword: "",
        redisMode: "cluster",
        image: "",
        create: true,
        machineIdList: [],
        machineList: [{ ip: "" }, { ip: "" }],
        autoBuild: true,
        autoInit: false,
        sudo: true,
        machineUserName: "",
        machinePassword: "",
        redisNodes: "127.0.0.1:8001 master\n127.0.0.1:8002",
        installationEnvironment: 0
      },
      installationInfoVisible: false,
      installationConsole: "Prepare to install redis...",
      rules: {
        clusterName: [
          {
            // required: true,
            validator: validateClusterName,
            trigger: "blur"
          },
          {
            // required: true,
            validator: validateClusterNameCurrentUsed,
            trigger: "blur"
          }
        ],
        // redisMode: [
        //   {
        //     required: true,
        //     message: "Please select redis mode",
        //     trigger: "change"
        //   }
        // ],
        installationEnvironment: [
          {
            required: true,
            message: "Please select installation environment",
            trigger: "change"
          }
        ],
        image: [
          {
            // required: true,
            message: "Please select image",
            trigger: "change"
          }
        ],
        machines: [
          {
            required: true,
            message: "Please select machines",
            trigger: "change"
          }
        ],
        // startPort: [
        //   {
        //     required: true,
        //     message: "Please enter start port",
        //     trigger: "blur",
        //
        //   },
        //   {
        //     required: true,
        //     validator: validateStartPort,
        //     trigger: "blur"
        //   }
        // ],
        // masterNumber: [
        //   {
        //     required: true,
        //     message: "Please eneter master number",
        //     trigger: "blur"
        //   },
        //   {
        //     type: "number",
        //     message: "Please enter integer",
        //     trigger: "blur"
        //   },
        //   {
        //     required: true,
        //     validator: validateMasterAndReplicaNumber,
        //     trigger: "blur"
        //   },
        //   {
        //     required: true,
        //     validator: validateMasterNumber,
        //     trigger: "blur"
        //   }
        // ],
        // replicaNumber: [
        //   {
        //     required: true,
        //     message: "Please eneter replica number",
        //     trigger: "blur"
        //   },
        //   { type: "number", message: "Please enter integer", trigger: "blur" },
        //   {
        //     required: true,
        //     validator: validateMasterAndReplicaNumber,
        //     trigger: "blur"
        //   }
        // ],
        machineUserName: [
          {
            required: true,
            message: "Please enter machine user name",
            trigger: "blur"
          }
        ],
        machinePassword: [
          {
            required: true,
            message: "Please enter machine password",
            trigger: "blur"
          }
        ],
        redisNodes: [
          {
            required: true,
            message: "Please enter redis topology",
            trigger: "blur"
          },
          {
            required: true,
            validator: validateTopology,
            trigger: "blur"
          }
        ]
      },
      allMachineList: [],
      // step: -1,
      installationLoading: false,
      humpbackEnabled: false,
      logTimer: null,
      isInstallationStart: false
    };
  },
  methods: {
    toDashboard() {
      this.installationLoading = false;
      message.success("Install Successfully");
      this.$router.push({
        name: "dashboard",
        params: { groupId: this.selectGroupId }
      });
    },
    handleMachine(val) {
      val.forEach(item => {
        // console.log(item[1]);
      });
    },
    buildMachineIdList() {
      this.installationParam.machineIdList = [];
      this.installationParam.machines.forEach(item => {
        this.installationParam.machineIdList.push(item[1]);
      });
    },
    buildParam() {
      let installationParam = this.installationParam;
      if (installationParam.autoBuild) {
        this.buildMachineIdList();
      }
      let cluster = {
        groupId: this.currentGroup.groupId,
        userId: store.getters.getUserId,
        clusterName: installationParam.clusterName,
        redisPassword: installationParam.redisPassword,
        redisMode: installationParam.redisMode,
        installationEnvironment: installationParam.installationEnvironment,
        image: installationParam.image,
        clusterInfo: installationParam.clusterInfo
      };
      this.installationParam.cluster = cluster;
    },
    installationCheck(installationParam) {
      this.$refs[installationParam].validate(valid => {
        if (valid) {
          //this.installationInfoVisible = true;
          this.buildParam();
          this.isInstallationStart = true;
          this.install();
        } else {
          return false;
        }
      });
    },
    install() {
      this.installationParam.masterNumber = this.installationParam.startPort.split(
        ","
      ).length;
      this.installationParam.replicaNumber = this.installationParam.replicaNumber.split(
        ","
      ).length;
      this.installationParam.replicaNumber =
        this.installationParam.replicaNumber /
        this.installationParam.masterNumber;
      this.installationParam.startPort = this.installationParam.startPort
        .split(",")
        .shift();
      this.installationLoading = true;
      // this.step = 0;
      let url = "/installation/installFlow";
      API.post(
        url,
        this.installationParam,
        response => {
          let result = response.data;
          console.log(result);
          if (result.code == 0) {
            this.$router.push({
              name: "dashboard",
              params: { groupId: this.currentGroup.groupId }
            });
          } else {
            message.error("Install failed");
          }
          this.installationLoading = false;
        },
        err => {
          this.installationLoading = false;
          message.error(err);
        }
      );
    },
    getDockerImageList() {
      let url = "/installation/getDockerImages";
      API.get(
        url,
        null,
        response => {
          let result = response.data;
          if (result.code == 0) {
            this.dockerImages = result.data;
          } else {
            message.error(result.message);
          }
        },
        err => {
          message.error(err);
        }
      );
    },
    getMachineImageList() {
      let url = "/installation/getMachineImages";
      API.get(
        url,
        null,
        response => {
          let result = response.data;
          if (result.code == 0) {
            this.machineImages = result.data;
          } else {
            message.error(result.message);
          }
        },
        err => {
          message.error(err);
        }
      );
    },
    getHumpbackImageList() {
      let url = "/installation/getHumpbackImages";
      API.get(
        url,
        null,
        response => {
          let result = response.data;
          if (result.code == 0) {
            this.humpbackImages = result.data;
          } else {
            message.error(result.message);
          }
        },
        err => {
          message.error(err);
        }
      );
    },
    getMachineList(groupId) {
      if (isEmpty(groupId)) {
        return;
      }
      let url = "/machine/getHierarchyMachineList/" + groupId;
      API.get(
        url,
        null,
        response => {
          let result = response.data;
          if (result.code == 0) {
            let hierarchyMachineList = result.data;
            hierarchyMachineList.forEach(oneMachineGroup => {
              let machineGroupName = oneMachineGroup.machineGroupName;
              oneMachineGroup.label = machineGroupName;
              oneMachineGroup.value = machineGroupName;
              let children = oneMachineGroup.children;
              children.forEach(machine => {
                machine.label = machine.host;
                machine.value = machine.machineId;
              });
              this.allMachineList = hierarchyMachineList;
            });
          } else {
            message.error(result.message);
          }
        },
        err => {
          message.error(err);
        }
      );
    },
    validateMachine(machine, handler) {},
    getHumpbackEnabled() {
      let url = "/system/humpbackEnabled";
      API.get(
        url,
        null,
        response => {
          let humpbackEnabled = response.data.data;
          if (humpbackEnabled) {
            this.humpbackEnabled = humpbackEnabled;
            this.getHumpbackImageList();
          }
        },
        err => {
          message.error(err);
        }
      );
    },
    getLogs() {
      if (this.isInstallationStart) {
        let url =
          "/installation/getInstallationLogs/" +
          this.installationParam.cluster.clusterName;
        API.get(
          url,
          null,
          response => {
            let logList = response.data.data;
            if (logList.length == 0) {
              return;
            }
            logList.forEach(log => {
              console.log(log);
              if (!isEmpty(log)) {
                if (log.indexOf("Start saving to database") > -1) {
                  this.step = 6;
                  console.log(6);

                  setTimeout(this.toDashboard, 1000);
                }
              }
            });
          },
          err => {
            message.error(err);
          }
        );
      }
    }
  },
  computed: {
    currentGroup() {
      return store.getters.getCurrentGroup;
    },
    installationEnvironmentList() {
      return store.getters.getInstallationEnvironmentList;
    }
  },
  watch: {
    "installationParam.autoBuild": function(newValue, oldValue) {
      let fields = this.$refs["installationParam"].fields;

      fields.map(field => {
        if (
          field.prop === "startPort" ||
          field.prop === "masterNumber" ||
          field.prop === "replicaNumber" ||
          field.prop === "machineUserName" ||
          field.prop === "machinePassword" ||
          field.prop === "topology" ||
          field.prop === "machines"
        ) {
          field.resetField();
          return false;
        }
      });
    },
    "installationParam.installationEnvironment": function(newValue, oldValue) {
      let fields = this.$refs["installationParam"].fields;
      fields.map(field => {
        if (field.prop === "image") {
          field.resetField();
          return false;
        }
      });
    },
    currentGroup(group) {
      let groupId = group.groupId;
      this.$router.push({
        name: "installation",
        params: { groupId: groupId }
      });
      this.getMachineList(groupId);
    }
  },
  mounted() {
    this.getDockerImageList();
    this.getMachineImageList();
    let groupId = this.currentGroup.groupId;
    this.getMachineList(groupId);
    this.getHumpbackEnabled();
    if (this.logTimer == null) {
      this.logTimer = setInterval(() => {
        this.getLogs();
      }, 1000);
    }
  },
  created() {
    clearInterval(this.logTimer);
    this.logTimer = null;
  },
  beforeDestroy() {
    clearInterval(this.logTimer);
    this.logTimer = null;
  }
};
</script>

<style scoped>
#installation {
  padding: 20px;
  background-color: #ffffff;
  height: 90%;
}

.step-wrapper {
  padding-top: 10px;
  padding-bottom: 10px;
  margin-bottom: 20px;
  border-bottom: 1px solid #dcdfe6;
}

.el-step__title {
  font-size: 14px !important;
}

.current-group {
  margin-bottom: 20px;
}

.form-wrapper {
  margin-right: 10px;
}

.info {
  color: #909399;
}

/* .console-wrapper {
  margin-left: 10px;
  border: 1px solid #dcdfe6;
  border-radius: 4px;
}

.console {
  height: 600px;
  padding: 10px 20px;
  background-color: black;
  color: #ffffff;
  font-family: Consolas, Monaco, Menlo, "Courier New", monospace !important;
  margin: 0;
  word-wrap: break-word;
  word-break: break-all;
  white-space: pre-wrap;
  line-height: 20px;
  overflow: auto;
}
.console-title {
  padding: 10px 20px;
  border-bottom: 1px solid #dcdfe6;
  background: #f0f2f5;
} */

.item-param {
  padding: 5px;
}

.param-key {
  color: #909399;
}
</style>
