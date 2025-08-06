<template>
  <div class="card-box">
    <a-list
      :grid="{ gutter: 24, lg: 2, md: 2, sm: 1, xs: 1 }"
      :dataSource="dataSource"
    >
      <a-list-item slot="renderItem" slot-scope="item">
        <template v-if="item.add">
          <div class="new-btn" @click="addColony({})">
            <div class="add-icon">
              <svg-icon
                icon-class="add-cluster"
                style="font-size: 80px"
              ></svg-icon>
            </div>
            <div class="add-text">创建集群</div>
          </div>
        </template>
        <template v-else>
          <div :class="['colony-card']">
            <div :class="['colony-status']">
              <div
                :class="[
                  'colony-status-text',
                  item.clusterStateCode === 2
                    ? 'running-status-text'
                    : item.clusterStateCode === 3
                    ? 'error-status-text'
                    : 'configured-status-text',
                ]"
              >
                <svg-icon
                  :class="[
                    'colony-status-icon',
                    item.clusterStateCode === 2
                      ? 'running-status-color-icon'
                      : item.clusterStateCode === 3
                      ? 'error-status-color'
                      : 'configured-status-color-icon',
                  ]"
                  :icon-class="
                    item.clusterStateCode === 2
                      ? 'running-status'
                      : 'configured-status'
                  "
                ></svg-icon>
                <span class="mgl5">{{ item.clusterState }}</span>
              </div>
              <a-button
                type="primary"
                @click="getInto(item)"
                :disabled="item.clusterStateCode === 1"
                >进入</a-button
              >
            </div>
            <div class="card-header flex-bewteen-container">
              <div class="flex-container">
                <div class="colony-title">{{ item.clusterName }}</div>
              </div>
            </div>
            <div class="card-content">
              <div>
                集群管理员：
                <span>{{ item.userManageName || "-" }}</span>
              </div>
              <div>
                创建时间：
                <span>{{ item.createTime }}</span>
              </div>
            </div>
            <div
              class="card-footer flex-bewteen-container"
              style="justify-content: start"
            >
              <a-button
                type="primary"
                v-if="user && user.userType === 1"
                @click="authCluster(item)"
                ghost
                >授权</a-button
              >
              <a-button
                type="primary"
                ghost
                @click="addColony(item)"
                :disabled="item.clusterStateCode === 2"
                >编辑</a-button
              >
              <a-button
                type="primary"
                ghost
                :disabled="item.clusterStateCode === 2"
                @click="configCluster(item)"
                >配置集群</a-button
              >
              <a-button
                @click="delectColony(item)"
                :disabled="item.clusterStateCode === 2"
                type="danger"
                ghost
                >删除集群</a-button
              >
            </div>
          </div>
        </template>
      </a-list-item>
    </a-list>
    <!-- 配置集群的modal -->
    <a-modal
      v-if="visible"
      title
      :visible="visible"
      :maskClosable="false"
      :closable="false"
      :width="1576"
      :confirm-loading="confirmLoading"
      @cancel="handleCancel"
      :footer="null"
    >
      <Steps :clusterId="clusterId" />
    </a-modal>
  </div>
</template>

<script>
import AddColony from "./commponents/addColony.vue";
import AuthCluster from "./commponents/authCluster.vue";
import DelectColony from "./commponents/delectColony.vue";
import { mapGetters, mapActions, mapMutations } from "vuex";
import Steps from "@/components/steps";
import { changeRouter } from "@/utils/changeRouter";
export default {
  name: "COLONYLIST",
  components: { Steps },
  provide() {
    return {
      handleCancel: this.handleCancel,
      onSearch: null,
    };
  },
  data() {
    return {
      visible: false,
      dataSource: [],
      confirmLoading: false,
      clusterId: "", // 操作的集群Id
    };
  },
  computed: {
    ...mapGetters("account", ["user"]),
  },
  methods: {
    ...mapMutations("setting", ["setIsCluster", "setMenuData", "setClusterId"]),
    // 进入
    getInto(row) {
      this.$axiosPost(global.API.getServiceListByCluster, {
        clusterId: row.id,
      }).then((res) => {
        changeRouter(res.data, row.id);
        this.$router.push("/overview");
      });
    },
    addColony(obj) {
      const self = this;
      let width = 520;
      let title = JSON.stringify(obj) !== "{}" ? "编辑集群" : "创建集群";
      let content = (
        <AddColony detail={obj} callBack={() => self.getColonyList()} />
      );
      this.$confirm({
        width: width,
        title: title,
        content: content,
        closable: true,
        icon: () => {
          return <div />;
        },
      });
    },
    delectColony(obj) {
      const self = this;
      let width = 400;
      let content = (
        <DelectColony
          sysTypeTxt="集群"
          detail={obj}
          callBack={() => self.getColonyList()}
        />
      );
      this.$confirm({
        width: width,
        title: () => {
          return (
            <div class="tips-title">
              <a-icon
                type="exclamation-circle"
                style="color:#F4622E !important;margin-right:10px"
              />
              提示
            </div>
          );
        },
        content,
        closable: true,
        icon: () => {
          return <div />;
        },
      });
    },
    getColonyList() {
      this.$axiosPost(global.API.getColonyList, {}).then((res) => {
        console.log(res);
        this.dataSource = res.data;
        this.dataSource.forEach((item) => {
          let arr = [];
          item.clusterManagerList.map((childItem) => {
            arr.push(childItem.username);
          });
          item["userManageName"] = arr.join(",");
        });
        this.dataSource.push({
          add: true,
        });
      });
    },
    // 集群授权
    authCluster(obj) {
      const self = this;
      let width = 520;
      let title = "授权";
      let content = (
        <AuthCluster detail={obj} callBack={() => self.getColonyList()} />
      );
      this.$confirm({
        width: width,
        title: title,
        content: content,
        closable: true,
        icon: () => {
          return <div />;
        },
      });
    },
    // 配置集群
    configCluster(row) {
      this.clusterId = row.id;
      this.setClusterId(row.id);
      this.visible = true;
    },
    handleCancel(e) {
      this.visible = false;
      this.getColonyList();
    },
  },
  mounted() {
    this.getColonyList();
  },
};
</script>

<style lang="less" scoped>
/deep/ .ant-modal-body {
  padding: 0;
}
/deep/ .ant-modal {
  top: 62px;
  .ant-modal-content {
    border-radius: 4px;
  }
}
.card-list {
  padding: 20px 10px;
  background: #fff;
  /deep/ .ant-row {
    margin: 0 !important;
  }
  /deep/ .ant-col {
    padding-left: 10px !important;
    padding-right: 10px !important;
  }
}
.colony-card {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  // cursor: pointer;
  height: auto;
  padding: 40px;
  background: #fff;
  border: 1px solid rgba(227, 228, 230, 1);
  border-radius: 4px;
  .card-header {
    .colony-icon-warp {
      width: 50px;
      height: 50px;
      border-radius: 50%;
      text-align: center;
      line-height: 50px;
      .colony-icon {
        // color: @primary-color;
        font-size: 24px;
        cursor: pointer;
      }
    }
    .colony-title {
      font-size: 24px;
      color: #333333;
      letter-spacing: 0;
      font-weight: 600;
    }
  }
  .colony-status {
    display: flex;
    justify-content: space-between;
    .colony-status-icon {
      font-size: 14px;
    }
  }
  .colony-status-text {
    height: 32px;
    width: auto;
    display: flex;
    align-items: center;
    padding: 4px 8px;
  }
  .running-status-text {
    background: rgba(116, 189, 106, 0.2);
  }
  .configured-status-text {
    background: rgba(246, 171, 40, 0.2);
  }
  .error-status-text {
    background: #fde0d5;
  }
  .card-content {
    div {
      margin-top: 20px;
      font-size: 14px;
      color: #73737f;
      letter-spacing: 0;
      font-weight: 400;
      span {
        color: #73737f;
        word-break: break-all;
        white-space: normal;
      }
    }
  }
  .card-footer {
    height: 50px;
    line-height: 50px;
    /deep/ .ant-btn-link {
      width: 20%;
      margin: 12px 0;
      border-radius: 0;
      font-size: 14px;
      color: #555555;
      letter-spacing: 0;
      font-weight: 400;
      border: none;
    }
    /deep/ .ant-btn-link:not(:last-child) {
      border: none;
      border-right: 1px solid#e3e4e6;
    }
    /deep/ .ant-btn-link:not(:last-child):hover,
    .ant-btn-link:not(:last-child):focus {
      border: none;
      border-right: 1px solid#e3e4e6;
    }
    /deep/ .ant-btn {
      margin-right: 10px;
      width: 120px;
      border-radius: 4px;
    }
  }
  /deep/ .ant-btn-link:not(.ant-btn-link[disabled]):hover {
    color: @primary-color;
  }
  /deep/ .ant-btn-link[disabled] {
    background: #fff;
    color: #bbb;
  }
}
.colony-running-card:hover {
  border: 1px solid @running-status-color;
}
.colony-configured-card:hover {
  border: 1px solid @configured-status-color;
}
.card-avatar {
  width: 48px;
  height: 48px;
  border-radius: 48px;
}
.new-btn {
  display: flex;
  flex-direction: column;
  justify-content: center;
  border-radius: 2px;
  width: 100%;
  height: auto;
  padding: 40px;
  border-radius: 2px;
  text-align: center;
  font-size: 16px;
  cursor: pointer;
  background: #fff;
  .add-icon {
    margin-top: 40px;
    // background-image: url("../../assets/img/colony/add-colony.svg");
    // background-size: 100% 100%;
    // margin: 0 auto 20px;
  }
  &:hover {
    color: @primary-color;
    // background: rgba(2, 121, 254, 0.03);
    // border: 1px dashed @primary-color;
    .add-icon {
      // background-image: url("../../assets/img/colony/add-colony-hover.svg");
    }
  }
}
.meta-content {
  position: relative;
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  height: 64px;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
}
.card-box {
  margin-top: 6px;
}
.flex-bewteen-container {
  margin-top: 20px;
}
.running-status-color-icon {
  color: #74bd6a !important;
}
.configured-status-color-icon {
  color: #f6ab28 !important;
}
.add-text {
  margin-top: 20px;
  height: 100px;
}
</style>
